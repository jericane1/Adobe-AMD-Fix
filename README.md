## Adobe Crash Fix for AMD

### Instructions

1. Install needed Adobe apps from Adobe Creative Cloud.

2. Patch Adobe Zii

3. Open Terminal.

4. Copy-paste the below command to your terminal and run it (enter password if asked).

```bash
files_list=(MMXCore FastCore TextModel libiomp5.dylib)
lib_dir="${HOME}/Documents/AdobeLibs"
lib1_file="${lib_dir}/libiomp5.dylib"
lib1_link="https://raw.githubusercontent.com/jericane1/Adobe-AMD-Fix/main/libiomp5.dylib"

for file in $files_list; do
    find /Applications/Adobe* -type f -name $file | while read -r curr_file; do
        name=$(basename $curr_file)
        sw_vers -productVersion | grep "11" >/dev/null 2>&1
        [[ $? ]] && [[ $name =~ ^(MMXCore|FastCore)$ ]] && continue
        echo "found $curr_file"
        sudo -v
        [[ ! -f ${curr_file}.back ]] && sudo cp -f $curr_file ${curr_file}.back || sudo cp -f ${curr_file}.back $curr_file
        if [[ $name == "libiomp5.dylib" ]]; then
            [[ ! -d $lib_dir ]] && mkdir $lib_dir
            [[ ! -f $lib1_file ]] && cd $lib_dir && curl -sO $lib1_link
            adobelib_dir=$(dirname "$curr_file")
            echo -n "replacing " && sudo cp -vf $lib1_file $adobelib_dir
        elif [[ $name == "TextModel" ]]; then
            echo "emptying $curr_file"
            sudo echo -n >$curr_file
        else
            echo "patching $curr_file"
            sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x6A\x00|\x90\x90\x90\x90\x56\xE8\x3A\x00|sg' $curr_file
            sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x4A\x00|\x90\x90\x90\x90\x56\xE8\x1A\x00|sg' $curr_file
        fi
    done
done
```
5. Now copy-paste the below command to terminal and run it (enter password if asked).

```bash
agent_dir="${HOME}/Library/LaunchAgents"
env_file="${agent_dir}/environment.plist"
lib_dir="${HOME}/Documents/AdobeLibs"
lib2_file="${lib_dir}/libintelfake.dylib"
lib2_link="https://raw.githubusercontent.com/jericane1/Adobe-AMD-Fix/main/libfakeintel.dylib"

sw_vers -productVersion | grep "11" >/dev/null 2>&1
if [[ $? ]]; then
    [[ ! -d $lib_dir ]] && mkdir $lib_dir
    [[ ! -f $lib2_file ]] && cd $lib_dir && curl -sO $lib2_link
    env="launchctl setenv DYLD_INSERT_LIBRARIES $lib2_file"
else
    mkl_value=$(
        sysctl -n machdep.cpu.brand_string | grep FX >/dev/null 2>&1
        echo $(($? != 0 ? 5 : 4))
    )
    env="launchctl setenv MKL_DEBUG_CPU_TYPE $mkl_value"
fi

[[ ! -d $agent_dir ]] && mkdir $agent_dir
cat >$env_file <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
 <key>Label</key>
 <string>mkl-debug</string>
 <key>ProgramArguments</key>
 <array>
 <string>sh</string>
 <string>-c</string>
    <string>$env;</string>
 </array>
 <key>RunAtLoad</key>
 <true/>
</dict>
</plist>
EOF

launchctl load ${AGENT} >/dev/null 2>&1
launchctl start ${AGENT} >/dev/null 2>&1
```
6. Now copy-paste the below command to terminal and run it (enter password if asked). Need ``` xcode-select —install ``` or [Command Line Tools for Xcode](https://gist.github.com/jericane1/1165f868123897f4a1ebe1c55d424f8b).

```
sudo codesign -fs - /Applications/Adobe*/Adobe*.app
```
7. - For Illustrator, copy-paste the below command to your terminal and run it (enter password if asked) : :
```
sudo rm -rf /Applications/Adobe\ Illustrator\ 2021/Adobe\ Illustrator\ 2021.app/Contents/Required/Plug-ins/Text\ Filters/TextModel.aip
```
- If Photoshop crashes while selecting font, copy-paste the below command to your terminal and run it (enter password if asked) :
```
sudo find /Applications/Adobe* -name "Deep_Font" -exec rm -r {} +
sudo rm -r /Applications/Adobe\ Photoshop\ 2020/Adobe\ Photoshop\ 2020.app/Contents/Required/Sensei_Models/Deep_Font
sudo rm -r /Applications/Adobe\ Photoshop\ 2021/Adobe\ Photoshop\ 2021.app/Contents/Required/Sensei_Models/Deep_Font
```
8. Reboot macOS.

### Revert Instructions

* To revert run the following command as required.
  * To revert step-3
```bash
files_list=(MMXCore FastCore TextModel libiomp5.dylib)
for file in $files_list; do
    find /Applications/Adobe* -type f -name $file | while read -r curr_file; do
        sudo -v
        [[ -f ${curr_file}.back ]] && echo "Restoring backup $curr_file"&& sudo mv -f ${curr_file}.back $curr_file
    done
done
```
  * To revert step-4
```bash
agent_dir="${HOME}/Library/LaunchAgents"
env_file="${agent_dir}/environment.plist"
if [[ -f $env_file ]]; then
    echo "Deleting $env_file"
    launchctl unload ${env_file} >/dev/null 2>&1
    launchctl stop ${env_file} >/dev/null 2>&1
    rm -rf $env_file
fi
```
2. Reboot macOS

### Application tested
>> Update on 19/08/2021

Adobe products | Version | Works ? | M1 ? (armv8) | Comments
---------------|---------|---------|--------------|---------
After Effects | 2021 v18.2 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
After Effects | 2021 v18.2.1 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
:new: After Effects | 2021 v18.4 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
Animate | 2021 v21.0.5 | :white_check_mark:| :x: | **No patch** Works, but Adobe ID ask login & password.
Animate | 2021 v21.0.6 | :white_check_mark:| :x: | **No patch** Works, but Adobe ID ask login & password.
Animate | 2021 v21.0.7 | :white_check_mark:| :x: | **No patch** Works, but Adobe ID ask login & password.
Audition | 2021 v14.1 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
Audition | 2021 v14.2 | :white_check_mark: | :white_check_mark: | **No patch** Works, no additional command lines.
:new: Audition | 2021 v14.4 | :white_check_mark: | :white_check_mark: |  **Apple Sillicon M1 works** ; **No patch** Works, no additional command lines. 
Bridge | 2021 v11.0 | :white_check_mark: | :x: | Works.
Bridge | 2021 v11.0.1 + CR 13.1 | :white_check_mark: | :x: | Works.
Bridge | 2021 v11.0.2 + CR 13.2 | :x: | :x: | **Zii problem !** On ignition, not start. Maybe it's Zii 6.1.0 rubbish ? With Adobe Creative Cloud, works perfectly.
Bridge | 2021 v11.1 + CR 13.3 | :white_check_mark: | :x: | Works. (Bridge & Camera Raw)
Character Animator | 2020 v4.0 | :white_check_mark: | :x: | **No patch** Works.
Character Animator | 2021 v4.2 | :white_check_mark: | :x: | **No patch** Works.
:new: Character Animator | 2021 v4.4 | :white_check_mark: | :white_check_mark: | **Apple Sillicon M1 works** Works.
Dimension | v3.4.1 | :white_check_mark: | :x: | Works.
Dimension | v3.4.2 | :white_check_mark: | :x: | Works.
Dimension | v3.4.3 | :white_check_mark: | :x: | Works. 
Dreamweaver | 2020 v20.2.1 | :white_check_mark: | :x: | Works.
Dreamweaver | 2021 v21.1 | :white_check_mark: | :x: | Works.
Illustrator | 2021 v25.2.0 | :x:| :x: | Illustrator works well but the Zii 6.x patch (with Big Sur) is really screwing up ... the application crashes right away. The Zii 6.x patch needs updating.
Illustrator | 2021 v25.3 | :white_check_mark:| :white_check_mark: | **Apple Sillicon M1 works** Works, no additional command lines.
Illustrator | 2021 v25.3.1 | :white_check_mark: | :white_check_mark: | **Apple Sillicon M1 works** Works, no additional command lines.
:new: Illustrator | 2021 v25.4 | :white_check_mark: | :white_check_mark: | **Apple Sillicon M1 works** Works, no additional command lines.
InCopy | 2021 v16.0.2 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InCopy | 2021 v16.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InCopy | 2021 v16.2 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InCopy | 2021 v16.3.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InDesign | 2021 v16.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InDesign | 2021 v16.2.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InDesign Server | 2021 v16.2.1 | :x: | :white_check_mark: | **Apple Sillicon M1 works**. Not works, need to Adobe ID !
InDesign | 2021 v16.3.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InDesign | 2021 v16.3.2 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
Lightroom Classic | v10.1 | :white_check_mark: | :x: | Works but you absolutely **Block Little Snitch or LuLu.**
Lightroom Classic | v10.1.1 | :white_check_mark: | :x: | Works you absolutely **Block Little Snitch or LuLu.**
Lightroom Classic | v10.2 | :interrobang: | :x: | Works perfectly, **but Camera Raw doesn't work. Perhaps Zii do not work, we do not know !** **Block Little Snitch or LuLu.**
Lightroom Classic | v10.3 | :interrobang: | :white_check_mark: | **Apple Sillicon M1 works** Works perfectly, **but Camera Raw doesn't work. Perhaps Zii do not work, we do not know !** **Block Little Snitch or LuLu.**.
Media Encoder | 2021 v15.1 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
Media Encoder | 2021 v15.2 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
:new: Media Encoder | 2021 v15.4 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
Photoshop | 2020 v21.2.3, v21.2.5, v21.2.10| :white_check_mark: | :x: | Works.
Photoshop | 2021 v22.0, **v22.4.2** | :x::interrobang: | :white_check_mark: | **Apple Sillicon M1 works** **Zii doesn't work. In addition, Intel inevitably works (Catalina, Big Sur, Monterey, etc.) and AMD (Ryzen, Threadripper) is failing at full speed! (nonstop crash)** Works  ... but it crashes all the time. We need to do more research for a patch. **No patch** Intel CPU : Works a charm. AMD CPU : Launch properly but crash all times.
:new: Photoshop | 2021 **v22.4.3** | :x::interrobang: | :white_check_mark: | **Apple Sillicon M1 works** ; Works, but when brushing or texting, no problem but when filters are put there, it crashes ! 
Prelude | 2020 v9.0.3 | :white_check_mark: | :x: | Works.
Prelude | 2021 v10.0 | :white_check_mark: | :x: | **No patch** Works.
:new: Prelude | 2021 v10.1 | :white_check_mark: | :x: | **No patch** Works.
Premiere Pro | 2021 v15.1 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
Premiere Pro | 2021 v15.2 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
:new: Premiere Pro | 2021 v15.4 | :white_check_mark: | :white_check_mark: | **Apple Sillicon M1 works** Works perfectly, no additional command lines.
Premiere Rush | v1.5.54 | :white_check_mark: | :x: | **No patch** Works.
Premiere Rush | v1.5.58 | :white_check_mark: | :white_check_mark: | **No patch** Works.
Premiere Rush | v1.5.61 | :white_check_mark: | :white_check_mark: | **No patch** Works.
Substance 3D Designer | v11.2 | :white_check_mark: | :x: | **No patch** Works.
:new: Substance 3D Designer | v11.2.1 | :white_check_mark: | :x: | **No patch** Works.
Substance 3D Painter | v7.2 | :x: | :x: | Crash ! 
Substance 3D Painter | v7.2.1 | :x: | :x: | Crash ! 
:new: Substance 3D Painter | v7.2.2 | :x: | :x: | Crash !
Substance 3D Sampler | v3.0 | :white_check_mark: | :x: | **No patch** Works.
:new: Substance 3D Sampler | v3.0.1 | :white_check_mark: | :x: | **No patch** Works.
:new: Substance 3D Stager | v1.0, **v1.0.1** | :white_check_mark: | :x: | **Problem Zii** : Copy the "vcontrol.bundle" in Zii 6.1.3 to the desktop:```Zii\ 2021\ 6.1.3.app/Contents/Resources/vcontrol.bundle``` Then rename it AdobeBIB.framework and replace it in ```/Applications/Adobe\ Substance\ 3D\ Stager/Adobe\ Substance\ 3D\ Stager.app/Contents/Frameworks/```.
XD | v41.0.12 | :white_check_mark: | :x: | **Block Little Snitch or LuLu.** Works, no additional command lines.
XD | v41.1.12 | :white_check_mark: | :x: | **Block Little Snitch or LuLu.** Works, no additional command lines.
:new: XD | v42.0.22 | :white_check_mark: | :white_check_mark: | **Apple Sillicon M1 works** ; **Block Little Snitch or LuLu.** Works, no additional command lines.
> Maybe the others need to be well controlled to be sure. Comments should help me! Thank you !

### Notes
- If you re-install any Adobe app then you will need redo the STEP-4 again.
