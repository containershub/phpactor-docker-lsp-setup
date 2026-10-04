# Phpactor Docker Setup
This is a minimal Phpactor Docker setup for PHP LSP running in Neovim with Mason.nvim

## WHY?
Why run Phpactor in a Docker container?

Short answer, I wanted to avoid installing PHP and Composer globally the same
way I'm avoiding installing NodeJS and NPM running on my local machine globally.
It's merely for security reasons. I want to keep my machines clean from anything
that can be used even by me to install something that I don't want, given the
fact that NPM was recently used as a convenient tool by hackers.
Sha1-Hulud! Remember that?!

Now that you know the WHY, let's now do the HOW!

### Step 1
* Clone this repository!
* Run `docker build -t local-phpactor:stable .` to build the image. The image
    will be called `local-phpactor:stable`.

After the image has been successfully built we've to setup a proxy script
inside Mason's `bin` directory.

### Step 2
* Run `nvim ~/.local/share/nvim/mason/bin/phpactor`
* Put the following shell code inside it:
    ```bash
    #!/bin/bash
    # Route Neovim LSP commands straight to our custom stable image
    docker run --rm -i \
      -v "$PWD":"$PWD" \
      -w "$PWD" \
      local-phpactor:stable "$@"
    ```
    This Docker command makes sure that the container is only up while Neovim
    is open, upon closing Neovim the container will no longer run and will be
    removed!
* Exit and run `chmod +x ~/.local/share/nvim/mason/bin/phpactor` to make
    executable.
* Next we've to create `mason-receipt.json` for Phpactor so Mason can recognize
    it and can run the script.
* Create a directory called `phpactor` inside `~/.local/share/nvim/mason/packages/`
    and file inside it called `mason-receipt.json` like:
    `~/.local/share/nvim/mason/packages/phpactor/mason-receipt.json`
* Now put the following JSON block inside it:
    ```json
    {
      "name": "phpactor",
      "links": {
        "bin": {
          "phpactor": "bin/phpactor"
        }
      },
      "metrics": {
        "completion_time": 0,
        "start_time": 0
      },
      "schema_version": "1.1.0"
    }
    ```
* Now you can open Neovim and run `:Mason` you'll see that `phpactor` is listed
    under `Installed` list!

### Step 3 - Test
Open a PHP file e.g. `nvim index.php` and you'll see that PHP diagnosis is
working code suggestions are available.

Also run `docker container ls` so you can see that `local-phpactor:stable` is
up and running. If you close Neovim and run `docker container ls` again you'll
see that that container is down!
