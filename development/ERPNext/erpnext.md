# Getting Started

## Bootstrap Containers for development

Clone and change directory to frappe_docker directory

```shell
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

Copy example devcontainer config from `devcontainer-example` to `.devcontainer`

```shell
cp -R devcontainer-example .devcontainer
```

Copy example vscode config for devcontainer from `development/vscode-example` to `development/.vscode`. This will setup basic configuration for debugging.

```shell
cp -R development/vscode-example development/.vscode
```

## Use VSCode Remote Containers extension

For most people getting started with Frappe development, the best solution is to use [VSCode Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).

Before opening the folder in container, determine the database that you want to use. The default is MariaDB.
If you want to use PostgreSQL instead, edit `.devcontainer/docker-compose.yml` and uncomment the section for `postgresql` service, and you may also want to comment `mariadb` as well.

VSCode should automatically inquire you to install the required extensions, that can also be installed manually as follows:

- Install Dev Containers for VSCode
  - through command line `code --install-extension ms-vscode-remote.remote-containers`
  - clicking on the Install button in the Vistual Studio Marketplace: [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  - View: Extensions command in VSCode (Windows: Ctrl+Shift+X; macOS: Cmd+Shift+X) then search for extension `ms-vscode-remote.remote-containers`

After the extensions are installed, you can:

- Open frappe_docker folder in VS Code.
  - `code .`
- Launch the command, from Command Palette (Ctrl + Shift + P) `Dev Containers: Reopen in Container`. You can also click in the bottom left corner to access the remote container menu.

Notes:

- The `development` directory is ignored by git. It is mounted and available inside the container. Create all your benches (installations of bench, the tool that manages frappe) inside this directory.
- Node v14 and v10 are installed. Check with `nvm ls`. Node v14 is used by default.

### Before setup bench run this command

It work in when you in devcontainer

```
sudo chmod 777 /workspace/development
```

### Setup first bench

> Jump to [scripts](#setup-bench--new-site-using-script) section to setup a bench automatically. Alternatively, you can setup a bench manually using below guide.

Run the following commands in the terminal inside the container. You might need to create a new terminal in VSCode.

NOTE: Prior to doing the following, make sure the user is **frappe**.

```shell
bench init --skip-redis-config-generation --frappe-branch v14.64.0 frappe-bench
cd frappe-bench
```

### Setup hosts

We need to tell bench to use the right containers instead of localhost. Run the following commands inside the container:

```shell
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379
```

For any reason the above commands fail, set the values in `common_site_config.json` manually.

```json
{
  "db_host": "mariadb",
  "redis_cache": "redis://redis-cache:6379",
  "redis_queue": "redis://redis-queue:6379",
  "redis_socketio": "redis://redis-queue:6379"
}
```

### Create a new site with bench

You can create a new site with the following command:

```shell
bench new-site excel_erpnext.localhost --mariadb-root-password 123 --admin-password admin --no-mariadb-socket
```

### Set bench developer mode on the new site

To develop a new app, the last step will be setting the site into developer mode. Documentation is available at [this link](https://frappe.io/docs/user/en/guides/app-development/how-enable-developer-mode-in-frappe).

```shell
bench --site excel_erpnext.localhost set-config developer_mode 1
bench --site excel_erpnext.localhost clear-cache
```

### Install all app

ERPNEXT

```shell
bench get-app --branch v14.60.1 erpnext
bench --site excel_erpnext.localhost install-app erpnext
```

HRMS

```shell
bench get-app --branch v14.21.0 hrms
bench --site excel_erpnext.localhost install-app hrms
```

EXCEL_ERPNEXT

```shell
bench get-app --branch version-14 https://gitlab.com/arcapps/excel_erpnext.git
bench --site excel_erpnext.localhost install-app excel_erpnext
```

WHITELABEL

```shell
bench get-app --branch main https://gitlab.com/excel.azmin/whitelabel.git
bench --site excel_erpnext.localhost install-app whitelabel
```

# After install all app

Run this bench

```shell
bench start
```

Browse this sites

```
http://excel_erpnext.localhost:8000/
```

# Restore Database

Download the `mariadb` database from [here](https://files-for-excel-bd.s3.ap-southeast-1.amazonaws.com/testerp+Oct4+post-migration/20221002_060134-testerp_excelbd_com-database.sql.gz). After complete the download extract file & past on frappe-bench directory.

- Create new `terminal` in your VS Code
- Run this command to restore database.

```
bench --site excel_erpnext.localhost --force restore 20211024_150012-testerp_excelbd_com-database.sql.gz
```

- It will take some time to successfully restore database.

**Brows your localhost**

```
http://excel_erpnext.localhost:8000/
```

**Login**

- Username : `shaid`
- Password : `Shaid@123`
