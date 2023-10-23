# Module tests

I rsynced over the whole /sw/mf folder, as well as all modules that are less than 50mb in size (keeping it small for testing), 293 of them. The names are stored in a file in this repo called `rackham_sw_bioinfo_less-than-50-mb.out`

```bash
# first define where the root of the UPPMAX-to-Dardel files should be
export UtDROOT=/cfs/klemming/projects/snic/naiss2023-22-1027/dahlo/testarea/uppmax_to_dardel

mkdir -p $UtDROOT/sw/bioinfo
cd $UtDROOT/sw/
rsync -a user@rackham.uppmax.uu.se:/sw/mf .

cd bioinfo
rsync -raP --files-from=rackham_sw_bioinfo_less-than-50mb.out user@rackham.uppmax.uu.se:/ .
```

I then replaced the paths containing `/sw/` in all the module files to point to the correct place in my test folder,

```bash
cd $UtDROOT/sw/mf/
find . -type f -exec sed -i -e "s%/sw/%$UtDROOT/sw/%g" {} \;
```

I also modified the function to get which cluster you are on so that it says `rackham` instead of `milou` as the default case (since dardel will not match anything in that file, and the most common location to install in in the rackham folder).

```bash
sed -i -e "s/set Cluster \"milou\"/set Cluster \"rackham\"/" $UtDROOT/sw/mf/common/includes/functions.tcl
```

Then you just have to source the `sourcreme.sh` file to get the right paths into your `MODULEPATH` variable and the module system at Dardel should see all the module files in `mf/`. It should be possible to see and load all modules, but only run the ones you actually copied over the binaries for.

```bash
source $UtDROOT/sourcreme.sh
module load bwa
bwa
```



**231019 update**

In a separate project, [uppmax_executables](https://github.com/NBISweden/uppmax_executables), all the shared libraries that were missing on Dardel was copied over from Rackham (placed in $UtDROOT/libs_from_rackham/) in hopes that they would just work. Initial test seem to be working, but we need to do actual test runs of more of the softwares to see if it really works.




## TODO


**Update /sw/ in all files**
We should probably replace all occurances of `/sw/` in all files in `sw/bioinfo` with the new path to the `sw` folder in this project, like we did for the module files above. Some scripts variables and hashbangs are hardcoded to the Rackham file structure and will not work when run on Dardel. Would probably look the same as the module find command above, and it is probably wise to figure out a way to exclude certain file types for this, e.g. .fasta, .hmm and the like, to avoid spending lots of time sed:ing data files that will never contain paths.

```bash
# like this, but with a file type filter defined
find . -type f -exec sed -i -e "s%/sw/%$UtDROOT/sw/%g" {} \;
```

**Remove loading of uppmax module**
Many module files will load the `uppmax` module while being loaded. There is no uppmax module at PDC so we either have to remove it from all module files, or create a mock module named `uppmax` that does nothing. The mock module sounds more robust to avoid sed:ing all module files again. Ok, i did it, stored in `$UtDROOT/sw/mf.as_on_rackham/rackham/bioinfo-tools/misc/uppmax`

**Remove loading of different gcc modules?**
We have versions of `gcc` on Rackham that does not exist on Dardel, e.g. `gcc/12.2.0`. Will stuff break if we remove all those loads? Or should they be changed to the Dardel equivalents?










