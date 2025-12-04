
# Splunk Lookup Uploader

To make it easier to upload lookup CSV files into Splunk remotely, I've cloned splunk_csv_importer and modified slightly. This utility will work from any python3 client environment, including Databricks.

## Features

- **Lookup CSV uploads**: Upload `.csv` files as Splunk lookups.
- **Backup Existing Lookups**: Automatically backup lookups with a timestamp if they already exist.

## Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/krdmnbrk/splunk_lookup_uploader.git
    cd splunk_lookup_uploader
    ```

2. **Install dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

3. **Set up `.env`**:
    ```ini
    # Splunk instance's host (usually localhost or a specific IP...just the host, not the full HTTP url)
    SPLUNK_HOST=

    # Splunk management port (usually 8089)
    SPLUNK_PORT=8089

    # Splunk token for authentication (if your installation has token auth enabled, favor this means to authenticate, otherwise leave it blank)
    SPLUNK_TOKEN=

    # Splunk username & password for authentication (favor using a non-human service user if you have one available)
    SPLUNK_USERNAME=
    SPLUNK_PASSWORD=

    # Unique delimiter for the SPL
    # You don't need to change this unless you have a specific requirement.
    # E.g if your data contains "|^|" you can change this to something else.
    UNIQUE_DELIMITER=|^|
    ```

## Usage

```bash
python splunk_lookup_uploader.py --source_file path/to/your.csv --target_lookup_name your_lookup_name.csv
```

Example:
```bash
python splunk_lookup_uploader.py --source_file your_lookup_file.csv --target_lookup_name your_lookup.csv
```

If a lookup exists, it will be backed up with a timestamp.  The lookup will have owner "nobody", associated with search application, and have global permissions.

## Example Workflow

1. Set up `.env` with your Splunk details.
2. Run the script with your CSV file.
3. Check your lookup in Splunk using:
    ```spl
    | inputlookup your_lookup_name.csv
    ```

**Note**: The script now checks for the presence of the config file and exits with a message if not found.  You should change the directory to the script directory (OR the directory with the .env file that you want to use) before running...like so:
```os.chdir("/directory/where/the/script/sits")```

## Databricks Particulars
To use from a Databricks notebook, download the zipped code locally, unzip, edit the .env file, then use the Databricks interface to upload the files to a folder in your Databricks workspace. In each notebook, you will need to add a !pip install --quiet -r requirements to the top of you notebook to add the splunk-sdk module for that session, unless you get your support team to install the splunk-sdk into the install as a permanently-available package. In the notebook, you can simply call the script using a **%run** magic command...the code below should work fine:
```bash
!pip install splunk-sdk==2.0.2
import os
os.chdir("/directory/where/this/script/lives")

%run splunk_lookup_uploader.py --source_file path/to/your_file.csv --target_lookup_name your_lookup.csv
```

## How It Works
An SPL search can be used to create a Splunk lookup from scratch, so this python script constructs that SPL using data from a CSV file, and then submits the SPL search to Splunk.  Here are examples that show the approach it's using:

First example:
```bash
| makeresults count=2
| eval this = case(_n=1, "this", _n=2, "that")
| eval that = case(_n=1, 123, _n=2, 456)
| fields this that
| outputlookup example_lookup.csv
```
Second example (hard-coded rows):
```bash
| makeresults
| eval data="this,123;that,456"
| makemv delim=";" data
| mvexpand data
| eval this=mvindex(split(data,","),0),
       that=mvindex(split(data,","),1)
| fields this that
| outputlookup example_lookup.csv
```

The python script builds the SPL with values from the lookup data, then submits it to Splunk similar to the examples immediately above. If the lookup already exists, it makes a backup with a date/time stamp.  This is a very simple approach that works reliably with a reasonable size of lookup data.
