# Google Sheets Fetcher

**A small Node.js foundation for accessing Google Sheets with a service account.**

Built with Express and `google-spreadsheet`, this project separates spreadsheet authentication from the HTTP layer. It is a starting point for turning spreadsheet content into data that other applications can consume.

## At a glance

- Authenticate with a Google service account.
- Load spreadsheet metadata and access worksheets through a reusable service.
- Use an Express server with CORS, JSON parsing, and centralized error responses.

> **Project status:** the spreadsheet service is implemented, but the `/stf` router has no request handler yet. The repository does not currently expose a working spreadsheet-fetching endpoint.

## Stack

Node.js · Express 4 · google-spreadsheet 3 · dotenv · CORS

## Getting started

```bash
git clone https://github.com/mateustalles/google-sheet-fetcher-nodejs.git
cd google-sheet-fetcher-nodejs
npm install
```

### 1. Configure Google access

Enable the Google Sheets API in your Google Cloud project, create a service account, and obtain its credentials. Share the target spreadsheet with the service account's email address; Viewer access is sufficient for reading.

Create a `.env` file in the project root:

```dotenv
GOOGLE_SERVICE_ACCOUNT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYOUR_KEY_HERE\n-----END PRIVATE KEY-----\n"
```

Keep credentials out of version control. Preserve the private key's line breaks when loading it from the environment.

### 2. Choose the spreadsheet

Replace the spreadsheet ID passed to `new GoogleSpreadsheet(...)` in [`services/spreadsheet.js`](services/spreadsheet.js).

The ID is the part between `/d/` and `/edit` in a Google Sheets URL:

```text
https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit
```

### 3. Use the service directly

The following example can be saved as `example.js` in the project root:

```js
require('dotenv').config();
const { fetchSpreadsheet } = require('./services/spreadsheet');

async function main() {
  const doc = await fetchSpreadsheet();
  const sheet = doc.sheetsByIndex[0];
  const rows = await sheet.getRows();

  console.log(`Spreadsheet: ${doc.title}`);
  console.log(`Worksheet: ${sheet.title}`);
  console.log(`Rows: ${rows.length}`);
}

main().catch(console.error);
```

```bash
node example.js
```

`fetchSpreadsheet()` returns the authenticated `GoogleSpreadsheet` instance after `loadInfo()`. Call `getRows()` on a worksheet to retrieve its rows; row access expects a header row.

## HTTP server

The entry point is `index.js`, and the server is configured to listen on port **3000**:

```bash
node index.js
```

Before using the HTTP server, implement the callback for `router.get('/')` in [`routers/stf.js`](routers/stf.js). Its current declaration has no callback and can prevent Express from starting.

The server mounts this router at `/stf`. Errors passed to the error middleware use this response shape:

```json
{
  "error": {
    "message": "Error description",
    "code": 500
  }
}
```

## Project structure

```text
index.js                 Express setup and error middleware
services/spreadsheet.js  Google Sheets authentication and metadata loading
routers/stf.js           HTTP route scaffold
models/connection.js     Unused MongoDB connection helper
```

## Development notes

- The listener uses a hardcoded port of `3000`; `PORT` only appears in the startup log.
- There is no `npm start` script. Use `node index.js` after completing the router.
- `npm test` is a placeholder that exits with an error.
- The MongoDB helper is not used by the entry point, and `mongodb` is not declared as a dependency. MongoDB is not required for the spreadsheet service.

## License

[MIT](LICENSE). The repository's license file specifies MIT; the `ISC` value in `package.json` is inconsistent with that file.
