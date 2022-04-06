# ESP-IDF Github Actions Build

This project is a simple example of how an ESP-IDF project can be automatically built using Github actions and the deployed to a github pages webpage with an embedded flasher system.

## See it in action
Link the this repo's github pages:
### [https://mawoodmain.github.io/esp-idf-actions/](https://mawoodmain.github.io/esp-idf-actions/)

## Build System
The project can be built using Espressif's first party github built system [link](https://github.com/espressif/esp-idf-ci-action)  
For this to work all you need to do is include a relevant workflow in `.github/workflows/<workflowname>.yml`  
On this project I created a work flow called main, [main.yml](./.github/workflows/main.yml) this file includes comments explaining how it works  

## Auto-Flasher Website
The work flow described above ([main.yml](./.github/workflows/main.yml)) copies the build output into the `pages` folder, this folder also has two files in it.  
- `index.html` This is the web page that will be hosted
- `manifest.json` This specifies what targets are supported and what files need to be flashed

The website [index.html](./pages/index.html) includes a script called `esp-web-tools` which can be found [here](https://github.com/esphome/esp-web-tools)  
This script adds the html tag `esp-web-install-button`, when called it is given with the `manifest.json` file above

### Turning on github pages
After the build process has run once, a new branch called `gh-pages` will exist, this hosts the web content for github pages, you need to configure github to use this branch, below is an image of this repos config: 
![Settings](./images/pages.png)
When you first set this up it may take a few minutes for the page to start working as github pages uses a CDN which adds a delay into the deployment process
### Writing the manifest.json file

The manifest json file is simply a reformatting of the build outputs `flash_args` file. The flash_args for this project looks like this:  
```
--flash_mode dio --flash_freq 40m --flash_size 2MB
0x0 bootloader/bootloader.bin
0x10000 hello_world.bin
0x8000 partition_table/partition-table.bin
```
This file describes the memory layout that needs to be programmed onto the target.  
In this case it says that `bootloader.bin` should be flashed starting at address `0`,  
`hello_world.bin` should be be flashed starting at address `0x10000` which is `65536` when represented in decimal as opposed to hexadecimal.  

The manifest.json file required for this project looks like this:
```json
{
  "name": "Hello World C3",
  "builds": [
    {
      "chipFamily": "ESP32-C3",
      "parts": [
        { "path": "build/bootloader/bootloader.bin", "offset": 0 },
        { "path": "build/hello_world.bin", "offset": 65536 },
        { "path": "build/partition_table/partition-table.bin", "offset": 32768 }
      ]
    }
  ]
}
```

Note that the offsets are specified in decimal, hex is not supported for numeric literals in json. Also, the paths have `build/` added as the build folder has been copied into the web directory.  
The name provided in this json is shown to the user, it can be customised as required.  
The chip family is also specified, multiple chip families can be specified however they do require different build files typically so you cannot just copy the build config and change the chip family.