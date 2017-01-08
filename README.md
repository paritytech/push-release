# push-release
## Push Parity releases to the chain

This is a PM2 node.js service which accepts simple RESTful-compliant HTTP requests as a trigger for updating an on-chain `Operations` contract with a new release.

## Specification

The two requests it can act upon are both POSTs (with params passed as URL-encoded data) and are at:

- `/push-release/<branch>/<commit>`: Pushes a the new release `commit` of `branch` . The `branch` should be one of `stable`, `beta`, `nightly` (equivalent to `master`) or `testing`. The `commit` should be the 40-digit hex-encoded Git commit hash of this release. A token must be supplied 
- `/push-build/<branch>/<platform>`: Pushes a single `platform`'s build of a release on `branch`. The `branch` is as above. The `platform` should be compliant according to `Operations` contract. The additional POST data is `commit` (as above), `filename` (the filename of the build in the build artefacts directory) and `sha3` (the hex-encoded Keccak-256 hash of the binary).

To ensure only valid updates are processed, all requests must provide an authentication token. The Keccak-256 hash of this token is stored in this script and any authentication tyoken which is passed must be the pre-image of this hash. It should be passed as a 64-digit, hex-encoded POST parameter with key `secret`.

### Options

- `account` The configuration for the account we use to post transactions. Should contain at most two keys: `address` and optionally `password`. If no password is supplied, it is assumed that the account is already unlocked.
- `baseUrl` The URL for artifacts; it will be appended with branch, platform and the filename, separated by directory delimiters (`/`s).
- `tokenHash` The hash of the secret. The pre-image to this hash must be provided by any requests.

## Deployment

We assume you have a preselected _signing account_ and _secret token_. You'll need to work out the Keccak-256 hash of the _secret token_ (you can use `require('js-sha3').keccak_256(secret_token)` to determine this).

1. Deploy `parity` on the host, ensuring it is set up to always by running on port 8545. Ensure your _signing account_ key is in the `parity` keys directory. Ensure your signing account is unlocked ready for use by creating a secured file containing its password and using the `--unlock` and `--password` parameters for `parity`.

2. Deploy Node.js, NPM and `pm2` on the host.

3. Clone `push-release` repository on the desired host:
```
git clone https://github.com/ethcore/push-release
```

4. Navigate in to the directory of push-release:
```
cd push-release
```

5. Edit `server.js`:

- Change the line beginning `const tokenHash =` to reflect the hash of the _secret token_ you created earlier.
- Change the line beginning `const account =` to have the `address:` of your `signing account` you decided on earlier. If you are not `--unlock`ing the account when running `parity`, you'll also need to provide the account's `password:`.
- Change the line beginning `const baseUrl =` to reflect the base URL of your build artefact's server address.

6. Install any required NPM modules:
```
npm install
```

7. Start the push-update service:
```
pm2 start push-release.json
```
