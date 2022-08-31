# postinst

`postinst` is a script for carrying out custom postinstallation tasks after an Ubuntu/Mint installation.

It takes one or more task names as its parameter:

```
postinst ungoogled-chromium

postinst natural-scrolling typecatcher
```

If run without parameters, it will run all tasks in interactive mode.

The list of available task names can be printed using the `--help` option.
The `-v` option will display the operations carried out by a task.
The `--dry-run` will just explain the steps but not execute them.

The operations for the tasks are taken from the `tasks.ini` file in the current directory.
Some of the tasks depend on the targeted desktop (like "lxqt") or release (like "jammy").
The desktop and release can be provided as command-line arguments.

There is one section per task in this file.
Each section has an action which describes the operation:

* `install_category`: Install a set of packages.

  The packages that are listed in the `all` key will be installed using `apt`.
  If the `desktop` key is set to true, also the packages under the corresponding desktop key will be installed.

  Example:

  ```
  [xbasics]
  action = install_category
  desktop = true
  all =
    xinput xsel xclip xdotool
    font-manager font-viewer xfonts-75dpi xfonts-100dpi
  lxqt = qt5-style-kvantum qt5-style-kvantum-themes
  xfce = xfce4-clipman-plugin
  ```

* `uninstall_category`: Uninstall a set of packages.

  The packages that are listed in the `all` key will be uninstalled using `apt`.
  If the `desktop` key is set to true, also the packages under the corresponding desktop key will be uninstalled.

  Example:

  ```
  [unwanted]
  action = uninstall_category
  desktop = true
  all = snapd
  ```

* `install_external`: Install a package from an external repository.

  The section name must match the name of the package.

  This action requires a `repo` key which is either a PPA, or the contents of a `/etc/apt/sources.list.d` entry.
  There can also be a `key` key as the contents of a `/etc/apt/trusted.gpg.d` entry.
  Both of these keys can be suffixed with the release name (as in `repo.jammy` or `key.jammy`)
  in order to restrain the use of that repo that that release.
  The file names for the sources and key entries will be generated using the package/section name.

  Example:

  ```
  [foliate]
  action = install_external
  repo = ppa:apandada1/foliate

  [element-desktop]
  action = install_external
  repo = deb https://packages.riot.im/debian/ default main
  key = https://packages.riot.im/debian/riot-im-archive-keyring.gpg
  ```

* `install_file`: Install a `.deb` file.

  This action requires a page URL that contains the download URL to the `.deb` file
  and a regex to extract that URL.

  The section name must match the name of the package.

  Example:

  ```
  [draw.io]
  action = install_file
  page = https://github.com/jgraph/drawio-desktop/releases
  regex = href="(.*(drawio-amd64-([0-9.]+)\.deb))"
  ```

  All of these actions are functions in the `postinst` script.
  Custom actions can be added to the script as functions and listed as actions.

  Example:

  ```
  [natural-scrolling]
  action = install_natural_scrolling
  ```

Tasks can be disabled by setting the `disabled` key to true.
If disabled, the task will be skipped in the script is run without specific tasks.
If the script is run with a specific task as parameter, the `disabled` key will have no effect.

All other keys will be ignored.
These can be used for note taking or other purposes.

Note: The actions are written so that running them multiple times will not cause any problems or repeated effects.
Custom action functions should also try to achieve this goal.
