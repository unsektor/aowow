# Run in Docker

Configure settings for first time:

1. Create environment settings files:
   ```sh
   cp .env.aowow.dist .env.aowow
   cp .env.mysql.dist .env.mysql
   ```
   Typically, `.env.mysql` is not required for production, because database server is hosted somewhere outside the docker.

   > [!NOTE]
   > You don't need to change `config/config.php` file, it will be overwritten with a special file, which
   > takes required credentials from the environment. See `docker/aowow/src/config/config.php` for details.
2. Edit `.env.*` files according to your environment, or leave as-is if environment is development.
3. Optionally: setup path to mpqdata for `aowow` service in `compose.dev.yaml` file:
   Replace this part of code:
   ```yaml
   # fixme optionally: change to directory with extracted mpq data
   - setup_aowow:/var/www/html/setup/mpqdata:rw
   # - /Users/md/Downloads/aowow-data/setup/mpqdata:/var/www/html/setup/mpqdata:ro
   ```
   with this (change `/.../aowow/setup/mpqdata` to path with your actual data):
   ```yaml
   - /.../aowow/setup/mpqdata:/var/www/html/setup/mpqdata:ro
   ```

Run application:

```sh
## for development:
docker compose up -f compose.yaml -f compose.dev.yaml

## for production:
docker compose up
```

> [!NOTE]
> First launch starts application initialization (database dumps download, loading dumps into database, downloading game data, extract media/info, etc ...)
> It may take long time (up to 30 minutes and more). Please, be patient.
>
> If it's required to skip initialization at first launch, then set `SETUP_SKIP` option value to `1`.

Then web-application should be accessible at <http://172.28.0.10>.

> [!NOTE]
> First launch may fail for some reason (e.g. fail to download data, slow network & healthcheck fail),
> it's recommended to rebuild containers for initialization scripts invocation.

Rebuild only aowow service (don't wipe database data):

```sh
docker container rm aowow-aowow-1
docker volume rm aowow_setup_aowow 
docker compose -f compose.yaml -f compose.dev.yaml up --build  
```

Full rebuild (wipe all data):

```sh
docker container rm aowow-mysql-1 aowow-aowow-1 
docker volume rm aowow_data_mysql aowow_data_aowow aowow_setup_aowow 
# docker volume rm aowow_setup_mysql  # ... contains trinitycore SQL dumps, typically it's never required to be removed 
docker compose -f compose.yaml -f compose.dev.yaml up --build  
```
