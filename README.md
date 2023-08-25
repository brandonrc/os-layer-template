# os-layer-template
Operating System Layer Template

## Background
There are many tools to use to configure an operating system. You can use ansible, packer, salt, puppet, and other fancy pants tools (bash scripts). Those tools did not help me create a custom image. I wanted something simple that can be used to make an image before it gets on the metal, physical computer/deployed virtual machine. 

This operating system layer template tries to mimic the design yocto accomplishes without requiring building everything and not including package managers. 

## How to use these layers? 
The idea is to include these layers as subs for your main operating system project/repo. The idea is to keep these layers as small as possible so you can reuse the layers with different projects. 

## File Structure


```
my-packer-build/
├── configs/ < --------------------------Configs folder is where you will put
│   ├── bin/                             custom config files that go in the /
│   │   └── custom-executable            root filesystem. Examples can include
│   ├── etc/                             custom dns, dhcpd, tftp, and other 
│   │   └── custom-executable.conf       services required for this "layer"
│   └── usr/local/bin
│       └── sylink-to-something
├── packages/   <------------------------Folder that contains lists for redhat
│   ├── rpms.txt                         and debian backages based on the flavor
│   ├── dpms.txt                         of linux. 
│   └── python.txt  <--------------------We also will include pip for the 
|                                        system python
├── scripts/
|   ├── 01-first-script-to-run.sh  <-----Scripts are run in ABC order.  
|   └── 02-second-script-to-run.sh       If you number them you can control
|                                        the order of the scripts
├── envionrment <------------------------Env variables added to the /etc/environment file
└── layer-dependencies <-----------------Any layers that are required to run this layer
```

## How can you use these layers? 
There are a few ways to use these layers while building an image. The perfered method would be to use packer and just include the layers in your build argument. 

### Packer THE EASY WAY
TODO...

### Alterative option (Custom tools) THE HARD WAY
Steps:
1. Packages
2. Configs
3. Scripts
4. Convert

Note: Order matters here so you might want to change how you are doing some commands. I run packages first because they will overwrite some `/etc` files along with `/var/` stuff as well. So I want those files installed first then I will just tar ontop with the configs we packed into a tar file with the instruction below. Then we will run the scripts after packages. Last we can convert that qcow2 image into a raw-img so we can `dd` to physical hardware. Or you can convert to vdi/vmdk if you choose for other VM platforms outside of `kvm`.

#### Packages
Example pseudo code:

```
package-list = ""
pip-list = ""
    for layer in layerdir:
        # Checking to see if we are on a redhat based linux system.
        if package manager is rpm:
            # We will add the list to the package list. 
            # Note: append is a fake function that magically
            # adds the contents of the file to the string list. 
            # Probably need to handle that differently based on 
            # python and bash. 
            # TODO: maybe add a scripting tool for help?
            package-list.append(layer + '/rpm-requirments.txt')
        else:
            package-list.append(layer + '/dpm-requirments.txt')
        # Same with pip packages
        pip-list.append(layer + '/pip-requirments.txt')

if rpm package manager:
    virt-customize --run-command "yum install -y package-list"
else:
    virt-customize --run-command "apt-get install -y package-list"

# pip install packages
# Be careful how you install pip packages.. system level pip packages can be dangerous and impact all users. You might want to install packages in a virt env. 
pip install pip-list
```  


Note: Again the above is pseudo code...


#### Configs
I have used python to merge all the config files from `n` layers into one `configs.tar.gz` and uploaded that to a `qcow2` file using `virt-customize`. 


#### Scripts
Then you can `for` loop the scripts and run those as seperate `virt-customize --run` commands. 


Example pseudo code:
```
for layer in layerdir:
    for scripts in scriptdir:
       `virt-customize --run scripts`

```

#### Convert
Then you can `qemu-convert -f qcow2 -O raw-img ./test.qcow2 ./test.img`

