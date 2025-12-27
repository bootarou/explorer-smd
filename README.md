# NFTDrive Symbol Explorer with SMD

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

NFTDrive Symbol Explorer is a read-only web application to browse the content of the Symbol blockchain.
The explorer supports searching for transactions, accounts, namespaces, mosaics, and blocks information on a given network.

## Key Features

### SMD (Social MetaData) Support
This explorer includes enhanced **Social MetaData (SMD)** functionality:
- Browse and search social metadata registered on Symbol blockchain
- View social profiles with icons, URLs, and namespace information
- Sort by name, namespace, URL, or registration order
- Multi-language support (Japanese/English)
- Direct navigation to account and namespace pages
- Real-time data fetching with Symbol blockchain integration

### Standard Symbol Explorer Features
The explorer supports searching for transactions, accounts, namespaces, mosaics, and blocks information on a given network.

## Requirements

- Node.js v20

## Installation

1. Clone the project.

```
git clone https://github.com/bootarou/explorer-smd.git
```

2. Install the required dependencies.

```
cd explorer-smd
npm install
```

3. Run the explorer application.

```
npm run dev
```

4. Visit http://localhost:8080/#/ in your browser.

## Developer notes

### Architecture

* `/src/config`: Handles the explorer configuration.
* `/src/infrastructure`: Handles the API / SDK request from Symbol nodes.
* `/src/store`: Handles the application logic with state management.
* `/src/views`: Handles the UI of the explorer.
* `/src/components/widgets`: Contains specialized components including SMD (Social MetaData) widget.

### SMD Implementation

* **MetadataService**: Fetches and parses social metadata from Symbol blockchain
* **SocialMetadataWidget**: Vue component for displaying social metadata cards
* **Multi-language support**: Japanese (ソーシャルメタデータ) and English translations
* **Navigation integration**: Direct links to accounts and namespaces
* **Sort functionality**: Multiple sorting options with ascending/descending order

## Getting help

Use the following available resources to get help:

- [Symbol Documentation][docs]
- Join the community [Discord][discord]
- If you found a bug, [open a new issue][issues]

## Contributing

Contributions are welcome and appreciated.
Check [CONTRIBUTING](CONTRIBUTING.md) for information on how to contribute.

## License

Copyright 2019-present NEM  
Copyright 2024-present NFTDrive

Licensed under the [Apache License 2.0](LICENSE)

This project is based on the original Symbol Explorer and enhanced with SMD functionality by NFTDrive.

[self]: https://github.com/bootarou/explorer-smd
[docs]: https://docs.symbolplatform.com
[issues]: https://github.com/bootarou/explorer-smd/issues
[discord]: https://discord.gg/NMA9YQ55td
