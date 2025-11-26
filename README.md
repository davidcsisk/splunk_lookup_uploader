
# Splunk Lookup Uploader

To make it easier to upload lookup CSV files into Splunk remotely, I've cloned and modified this python app.
've
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

If a lookup exists, it will be backed up with a timestamp.  The lookup will have owner "nobody", associated with search application, and global permissions.

## Example Workflow

1. Set up `.env` with your Splunk details.
2. Run the script with your CSV file.
3. Check your lookup in Splunk using:
    ```spl
    | inputlookup your_lookup_name.csv
    ```

## Contributions

Feel free to fork or submit PRs. I built this to solve a problem, and I'm happy if it helps others!
