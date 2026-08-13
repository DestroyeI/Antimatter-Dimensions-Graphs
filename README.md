# Antimatter Dimensions

## Play the Game

The best way to run the game is through the public
[**Github Pages**](https://destroyei.github.io/Antimatter-Dimensions-Graphs/) link.
This will allow you to simply run the modified game in your browser with no setup.

## Running a Local Server **(UNRECOMMENDED)**

To run the game locally, you will need to install
[Node.js](https://nodejs.org/) (LTS suggested).

First, run the following command in your terminal (or command line) while being
inside the checked out repository:

```
npm ci
```

After all the packages are installed, start up the game:

```
npm run serve
```

This will make the game served via your localhost, and the playable link will
be displayed in your terminal. The server **doesn't** need to be restarted
after you've made changes - just reload the page. The server **can**
occasionally crash, so check your terminal from time to time and run `serve`
again if needed.

This option is only helpful if you plan to **clone and modify** this repository.
