# docker-compose-laravel-statamic
A pretty simplified Docker Compose workflow that sets up a LEMP network of containers for local Laravel development. [Forked from here](https://github.com/aschmelyun/docker-compose-laravel) and tweaked for Windows and my needs (namly, building a stable [Statamic](https://statamic.dev/) development environment) - thank you!

## Prerequisites for Windows and Workflow

I'm using this repo on a Windows environment, which makes for an interesting and slightly quirky workflow. You will need to be aware of and action the following:

- [Install Docker](https://docs.docker.com/desktop/windows/install/) and activate the WSL 2 based engine
- Install Ubuntu (via Microsoft Store) and I also recommend using [Windows Terminal](https://aka.ms/terminal) (it's pretty good!)
- Activate Ubuntu in the Docker Settings > Resources > WSL Integration
- Optionally create a syslink Projects folder in Ubuntu which you can also access from your Windows host machine. From your home directory in Ubuntu, something like `ln -s /mnt/c/Users/<username>/Documents/Projects Projects`. Via windows explorer use \\wsl$\ enter the virtual machine file system.
- I'm using VS code opened from within Ubuntu via Windows Terminal using `code .` in your projects folder.

If you need some help understanding more about WSL 2 and how it works, I recommend watching a few videos from [Beachcasts Programming Videos](https://www.youtube.com/channel/UCsOSGYawy8MG9Mh8NKgRHZQ).

## Install Lavarel

Follow the steps from the [src/README.md](src/README.md) file to get your Laravel project added in (or create a new blank one).

## Install Statamic

Following the [instructions here](https://statamic.dev/installing/laravel) run `docker-compose run composer require statamic/cms --updat
e-with-all-dependencies`.

## Usage

Next, navigate in your terminal to the directory you cloned this, and spin up the containers for the web server by running `docker-compose up -d --build site`.

After that completes, follow the steps from the [src/README.md](src/README.md) file to get your Laravel project added in (or create a new blank one).

Bringing up the Docker Compose network with `site` instead of just using `up`, ensures that only our site's containers are brought up at the start, instead of all of the command containers as well. The following are built for our web server, with their exposed ports detailed:

- **nginx** - `:80`
- **mysql** - `:3306`
- **php** - `:9000`
- **redis** - `:6379`
- **mailhog** - `:8025` 

Three additional containers are included that handle Composer, NPM, and Artisan commands *without* having to have these platforms installed on your local computer. Use the following command examples from your project root, modifying them to fit your particular use case.

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## Permissions Issues

If you encounter any issues with filesystem permissions while visiting your application or running a container command, try completing one of the sets of steps below.

**If you are using your server or local environment as the root user:**

- Bring any container(s) down with `docker-compose down`
- Rename `docker-compose.root.yml` file to `docker-compose.root.yml`, replacing the previous one
- Re-build the containers by running `docker-compose build --no-cache`

**If you are using your server or local environment as a user that is not root:**

- Bring any container(s) down with `docker-compose down`
- In your terminal, run `export UID=$(id -u)` and then `export GID=$(id -g)`
- If you see any errors about readonly variables from the above step, you can ignore them and continue
- Re-build the containers by running `docker-compose build --no-cache`

Then, either bring back up your container network or re-run the command you were trying before, and see if that fixes it.

## Persistent MySQL Storage

By default, whenever you bring down the Docker network, your MySQL data will be removed after the containers are destroyed. If you would like to have persistent data that remains after bringing containers down and back up, do the following:

1. Create a `mysql` folder in the project root, alongside the `nginx` and `src` folders.
2. Under the mysql service in your `docker-compose.yml` file, add the following lines:

```
volumes:
  - ./mysql:/var/lib/mysql
```

## Using BrowserSync with Laravel Mix

If you want to enable the hot-reloading that comes with Laravel Mix's BrowserSync option, you'll have to follow a few small steps. First, ensure that you're using the updated `docker-compose.yml` with the `:3000` and `:3001` ports open on the npm service. Then, add the following to the end of your Laravel project's `webpack.mix.js` file:

```javascript
.browserSync({
    proxy: 'site',
    open: false,
    port: 3000,
});
```

From your terminal window at the project root, run the following command to start watching for changes with the npm container and its mapped ports:

```bash
docker-compose run --rm --service-ports npm run watch
```

That should keep a small info pane open in your terminal (which you can exit with Ctrl + C). Visiting [localhost:3000](http://localhost:3000) in your browser should then load up your Laravel application with BrowserSync enabled and hot-reloading active.

## MailHog

The current version of Laravel (8 as of today) uses MailHog as the default application for testing email sending and general SMTP work during local development. Using the provided Docker Hub image, getting an instance set up and ready is simple and straight-forward. The service is included in the `docker-compose.yml` file, and spins up alongside the webserver and database services.

To see the dashboard and view any emails coming through the system, visit [localhost:8025](http://localhost:8025) after running `docker-compose up -d site`.
