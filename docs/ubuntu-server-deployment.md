# Ubuntu Server Deployment (Headless)

This guide covers a first production deployment on a headless Ubuntu Server when the operator performs the initial Google OAuth authorization on a desktop computer and manually transfers the resulting credential files to the server.

The recommended approach keeps the existing **Desktop app OAuth client** and the current `auth` implementation. No browser, public callback endpoint, Web OAuth client, or application code change is required on the VPS.

## Deployment Model

The service authenticates as the YouTube channel owner. The initial authorization is interactive, but production `run` is non-interactive:

1. Prepare the application, configuration, FFmpeg, and Google OAuth Desktop client on the development PC.
2. Run `auth` on that PC and complete Google's browser consent flow.
3. Securely transfer **both** OAuth files to the VPS:
   - `credentials/client_secret.json` — the Desktop app client configuration.
   - `credentials/tokens.json` — the issued access/refresh credentials.
4. Install the Linux release artifact and its runtime assets on the VPS.
5. Configure the server's paths and permissions, then run `run`.

The client file is required as well as the token file: the application needs the OAuth client configuration when it refreshes an access token. Transferring only `tokens.json` is not a complete provisioning procedure.

The OAuth grant remains bound to the YouTube channel selected during the desktop authorization. Refreshing it on the VPS does not select or switch channels.

## 1. Prepare and Authorize on the Desktop PC

Use the same Google Cloud project and OAuth **Desktop app** client intended for the server.

1. Follow [Google Cloud prerequisites](google-cloud-prerequisites.md) and [Authentication](authentication.md) to configure the project and client.
2. Ensure the local configuration uses distinct paths for:
   - `oauth.client_config_path` (default: `credentials/client_secret.json`);
   - `oauth.credentials_path` (default: `credentials/tokens.json`).
3. Run the current application build:
   
   ```sh
   rust-youtube-streamer-service auth
   ```

4. Open the printed authorization URL in a browser, sign in as the account/channel owner, select the intended channel if Google prompts for one, and approve the requested YouTube permission.
5. Confirm that `auth` exits successfully and that both files exist.

Do not copy the authorization URL, authorization code, access token, or refresh token into notes, tickets, shell history, or chat. Do not commit either credential file.

## 2. Prepare the Ubuntu VPS

Use a dedicated, non-root Linux account for the service. The examples below use `ytstream`; replace it with the account and paths chosen for your server.

Create the account and application directories:

```sh
sudo useradd --system --create-home --home-dir /opt/ytstream --shell /usr/sbin/nologin ytstream
sudo install -d -o ytstream -g ytstream -m 0750 /opt/ytstream/app
sudo install -d -o ytstream -g ytstream -m 0700 /opt/ytstream/credentials
sudo install -d -o ytstream -g ytstream -m 0750 /opt/ytstream/config
sudo install -d -o ytstream -g ytstream -m 0750 /opt/ytstream/logs
sudo install -d -o ytstream -g ytstream -m 0750 /opt/ytstream/videos
```

Install FFmpeg from the Ubuntu package source or another trusted distribution channel appropriate for the server:

```sh
sudo apt update
sudo apt install ffmpeg
ffmpeg -version
ffmpeg -hide_banner -encoders
```

The configured FFmpeg executable must be runnable by the service account and include the encoders required by the selected streaming configuration. In particular, when silent audio is enabled, the build must provide `libmp3lame`. FFmpeg is an external runtime prerequisite; the project does not bundle or install it.

Copy the Linux release artifact and the runtime assets it needs (including the configured font) into the deployment layout. Keep the executable, configuration, font, logs, and output directories in stable, documented locations. Do not assume that a development-machine path exists on Ubuntu.

## 3. Transfer the OAuth Files Securely

From the desktop PC, use an authenticated SSH/SFTP connection to copy the files to the VPS. For example, if the local files are in the repository's `credentials/` directory:

```sh
scp credentials/client_secret.json credentials/tokens.json \
  your-ssh-user@your-server:/tmp/
```

Then, on the VPS, move them into the service-owned credential directory and restrict access:

```sh
sudo install -o ytstream -g ytstream -m 0600 \
  /tmp/client_secret.json /opt/ytstream/credentials/client_secret.json
sudo install -o ytstream -g ytstream -m 0600 \
  /tmp/tokens.json /opt/ytstream/credentials/tokens.json
rm -f /tmp/client_secret.json /tmp/tokens.json
```

Use a secure transfer path and remove temporary copies from both machines when they are no longer needed. The commands above assume the files are staged in `/tmp`; a more restrictive temporary directory is preferable on a multi-user server.

Verify ownership and permissions without printing file contents:

```sh
sudo stat -c '%U:%G %a %n' /opt/ytstream/credentials/*
```

Expected permissions are owner `ytstream`, group `ytstream`, and mode `600` for each file. The application also writes refreshed credentials atomically, so the service account must retain write permission on the credential directory.

**Never** put credential JSON in the repository, a public web directory, a container image, a deployment artifact, or a command-line argument. Do not print either file to the terminal or include its contents in logs.

## 4. Configure the Server

Create the server's real configuration from the committed example, then edit it for the Linux deployment:

```sh
sudo install -o ytstream -g ytstream -m 0640 \
  config/config.example.json /opt/ytstream/config/config.json
sudoedit /opt/ytstream/config/config.json
```

Set the OAuth paths to the server locations:

```json
{
  "oauth": {
    "client_config_path": "/opt/ytstream/credentials/client_secret.json",
    "credentials_path": "/opt/ytstream/credentials/tokens.json"
  }
}
```

The fragment above illustrates only the `oauth` section; preserve all other required top-level configuration sections from `config.example.json`. Keep the two OAuth paths distinct.

Review every path in the full configuration, especially:

- `renderer.font`: an existing, readable font file on the VPS;
- `ffmpeg.executable`: usually `/usr/bin/ffmpeg` when installed from Ubuntu packages;
- `chat.log_file`: a writable path for the service account, preferably under `/opt/ytstream/logs/`;
- diagnostics output paths, if any diagnostic file features are enabled;
- video dimensions, FPS, and FFmpeg settings appropriate for the server's available resources.

Use absolute paths for server deployments to avoid ambiguity about the process working directory. Ensure the service account can read the configuration, font, executable, and credentials, and can write the chat log, refreshed token file, and any enabled diagnostic/output files.

The configuration file may contain operational details even when it contains no OAuth tokens. Keep it out of version control and restrict its permissions.

## 5. Run and Verify on the VPS

Run the service as the dedicated account, using the deployed executable and explicit configuration path:

```sh
sudo -u ytstream /opt/ytstream/app/rust-youtube-streamer-service \
  --config /opt/ytstream/config/config.json run
```

The exact executable path depends on the release layout you install.

On startup, the credential service loads the persisted grant, verifies it with the YouTube API, and refreshes once if verification rejects the access token and a refresh token is available. A successful refresh is persisted atomically. The `run` command does not launch a browser or prompt for interactive input.

For the first server run, observe stderr and confirm the documented startup sequence in [Production Run](production-run.md), including broadcast preparation, FFmpeg startup, stream confirmation, and Live Chat initialization. Confirm in YouTube Studio that the broadcast belongs to the intended channel. Perform the operator-run live verification procedure before treating the VPS deployment as production-validated.

Do not infer that the complete end-to-end YouTube path is verified merely because the process starts or the OAuth probe succeeds. Native Ubuntu runtime and real-broadcast validation remain operator-run checks unless separately recorded.

## 6. OAuth Token Lifetime and Google Cloud Publishing Status

**Important: the OAuth consent screen's publishing status affects unattended operation.**

When a Google OAuth app has user type **External** and publishing status **Testing**, refresh tokens issued to test users generally expire after **7 days** when the app requests scopes beyond the basic identity scopes. This project requests the YouTube `youtube.force-ssl` scope, so the basic-identity exception does not apply. A server that runs unattended can therefore stop refreshing successfully after the testing token expires, even when the files and permissions are correct.

Before relying on a long-running VPS deployment:

1. Check the app's current publishing status in Google Auth Platform → Audience.
2. Do not treat **Testing** as a permanent unattended-production configuration for this YouTube scope.
3. Evaluate moving the app to **In production** and review Google's current verification requirements for the sensitive YouTube scope. Publishing status and OAuth verification are related but distinct: changing the status does not itself mean that the scope has been verified or that every warning/restriction is removed.
4. Complete any verification or policy steps Google requires for the intended use. Re-authorize after changing the app's OAuth publishing/consent configuration if Google requires a fresh grant or if existing credentials remain unusable.

An unverified app may display an authorization warning and can be subject to user caps or other restrictions. Do not assume that an unverified sensitive-scope app is suitable for unrestricted public distribution. Consult the current Google documentation linked below and the status shown in your own Cloud project.

The application cannot override Google's token-expiration, revocation, or OAuth publishing policies. It can refresh only while Google accepts the stored refresh token.

## 7. Recovery and Credential Rotation

If `run` reports missing, malformed, revoked, expired, or insufficient credentials:

1. Re-check that both files exist at the configured paths and are readable by `ytstream`.
2. Check the Google Cloud OAuth app's publishing status and any consent-screen restrictions.
3. If Google rejected the refresh token (for example, `invalid_grant`), perform a fresh `auth` flow on the desktop PC.
4. Transfer both the newly issued `client_secret.json` and `tokens.json` to the VPS using the secure procedure above, replacing the old files with owner-only permissions.
5. Restart the service and verify the startup sequence.

Do not attempt to repair a revoked refresh token by editing JSON, copying an access token alone, or manually constructing token fields. Re-run the supported authorization flow.

If the OAuth client is replaced or rotated, transfer the matching new client configuration and authorize again as needed. Treat old credential files as compromised if they were exposed; revoke the grant through the relevant Google account/security controls and provision a fresh one.

## 8. Operational Security Checklist

- [ ] Service runs as a dedicated non-root Linux user.
- [ ] Both OAuth files are present on the VPS and have mode `0600`.
- [ ] The service account owns the credential directory and can atomically replace `tokens.json`.
- [ ] Both OAuth paths are absolute, readable, distinct, and outside the repository.
- [ ] Real configuration is not committed and has restricted permissions.
- [ ] FFmpeg is installed separately and supports the configured encoders.
- [ ] Font, chat log, diagnostics, and output paths exist or can be created with the required permissions.
- [ ] OAuth publishing status has been reviewed; the 7-day Testing expiry is not being mistaken for a code defect.
- [ ] Broadcast channel identity is confirmed in YouTube Studio.
- [ ] A real Ubuntu + YouTube operator-run verification has been completed and recorded.
- [ ] Backups of credential files, if made, are encrypted and access-restricted; old copies are securely removed when retired.

## See Also

- [Authentication](authentication.md) — OAuth flow, credential schema, refresh behavior, and security guarantees.
- [Google Cloud Prerequisites](google-cloud-prerequisites.md) — project, YouTube Data API, OAuth client, scope, and channel prerequisites.
- [Configuration](configuration.md) — full configuration schema and validation.
- [Production Run](production-run.md) — broadcast lifecycle, Live Chat, diagnostics, shutdown, and operator verification.
- [Linux Release Build](../README.md#release-build-linux-artifact) — building the Linux executable.
- [Google OAuth app publishing status](https://support.google.com/cloud/answer/15549945).
- [Google OAuth verification requirements](https://support.google.com/cloud/answer/13463073).
- [OAuth 2.0 for installed applications](https://developers.google.com/identity/protocols/oauth2/native-app).
