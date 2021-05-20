
## Adobe Crash Fix for AMD
Adobe Creative Cloud fix for AMD Ryzen in macOS 

### Instructions

1. Install needed Adobe apps from Adobe Creative Cloud.

2. Patch Adobe Zii

3. Open Terminal.

4. Copy-paste the below command to your terminal and run it (enter password if asked).

```
for file in MMXCore FastCore TextModel libiomp5.dylib libtbb.dylib libtbbmalloc.dylib; do
    find /Applications/Adobe* -type f -name $file | while read -r FILE; do
        sudo -v
        echo "found $FILE"
        [[ ! -f ${FILE}.back ]] && sudo cp -f $FILE ${FILE}.back || sudo cp -f ${FILE}.back $FILE
        echo $FILE | grep libiomp5 >/dev/null
        if [[ $? == 0 ]]; then
            dir=$(dirname "$FILE")
            [[ ! -f ${HOME}/libiomp5.dylib ]] && cd $HOME && curl -sO https://github.com/jericane1/Adobe-AMD-Fix/raw/main/libiomp5.dylib
            echo -n "replacing " && sudo cp -vf ${HOME}/libiomp5.dylib $dir && echo
            rm -f ${HOME}/libiomp5.dylib
            continue
        fi
        echo $FILE | grep TextModel >/dev/null
        [[ $? == 0 ]] && echo "emptying $FILE" && sudo echo -n >$FILE && continue
        echo "patching $FILE \n"
        sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x6A\x00|\x90\x90\x90\x90\x56\xE8\x3A\x00|sg' $FILE 
        sudo perl -i -pe 's|\x90\x90\x90\x90\x56\xE8\x4A\x00|\x90\x90\x90\x90\x56\xE8\x1A\x00|sg' $FILE 
    done
done
```
5. Now copy-paste the below command to terminal and run it (enter password if asked).

```
[ ! -d $HOME/Library/LaunchAgents ] && mkdir $HOME/Library/LaunchAgents
AGENT=$HOME/Library/LaunchAgents/environment.plist
sysctl -n machdep.cpu.brand_string | grep FX >/dev/null 2>&1
x=$(echo $(($? != 0 ? 5 : 4)))
cat >$AGENT <<EOF
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
    <string>launchctl setenv MKL_DEBUG_CPU_TYPE $x;</string>
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
7. For Illustrator only :
```
sudo rm -rf /Applications/Adobe\ Illustrator\ 2021/Adobe\ Illustrator\ 2021.app/Contents/Required/Plug-ins/Text\ Filters/TextModel.aip
```
8. Reboot macOS.

PS : If Photoshop crashes while selecting font, copy-paste the below command to your terminal and run it (enter password if asked).
```
sudo rm -r /Applications/Adobe\ Photoshop\ 2020/Adobe\ Photoshop\ 2020.app/Contents/Required/Sensei_Models/Deep_Font
sudo rm -r /Applications/Adobe\ Photoshop\ 2021/Adobe\ Photoshop\ 2021.app/Contents/Required/Sensei_Models/Deep_Font
```
### Revert Instructions

1. To revert run the following command in terminal.
```
for file in MMXCore FastCore TextModel libiomp5.dylib libtbb.dylib libtbbmalloc.dylib; do
    find /Applications/Adobe* -type f -name $file | while read -r FILE; do
        sudo -v
        [[ -f ${FILE}.back ]] && echo "found backup $FILE" && sudo mv -f ${FILE}.back $FILE
    done
done

AGENT=$HOME/Library/LaunchAgents/environment.plist
if [[ -f $AGENT ]]; then
    launchctl unload ${AGENT} >/dev/null 2>&1
    launchctl stop ${AGENT} >/dev/null 2>&1
    rm -rf $AGENT
fi
```
2. Reboot macOS

### Application tested
>> Update on 18/05/2021

Adobe products | Version | Works ? | M1 ? (armv8) | Comments
---------------|---------|---------|--------------|---------
After Effects | 2021 v18.0 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
After Effects | 2021 v18.1 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
:new: After Effects | 2021 v18.2 | :question: | :question: | ???
Animate | 2021 v21.0.4 | :white_check_mark: | :x: | Works.
Animate | 2021 v21.0.5 | :white_check_mark:| :x: | **No patch*** Works, but Adobe ID ask login & password.
:new: Animate | 2021 v21.0.6 | :question:| :question: | ???
Audition | 2021 v14.0 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
Audition | 2021 v14.1 | :white_check_mark: | :x: | **No patch** Works, no additional command lines.
:new: Audition | 2021 v14.2 | :question: | :question: | ????
Bridge | 2021 v11.0 | :white_check_mark: | :x: | Works.
Bridge | 2021 v11.0.1 + CR 13.1 | :white_check_mark: | :x: | Works.
Bridge | 2021 v11.0.2 + CR 13.2 | :x: | :x: | On ignition, not start. Maybe it's Zii 6.1.0 rubbish ? With Adobe Creative Cloud, works perfectly.
Character Animator | 2020 v3.5 | :white_check_mark: | :x: | Works.
Character Animator | 2020 v4.0 | :white_check_mark: | :x: | **No patch** Works.
:new: Character Animator | 2021 v4.2 | :question: | :question: | ????
Dimension | v3.4.1 | :white_check_mark: | :x: | Works.
:new: Dimension | v3.4.2 | :question: | :question: | ????
Dreamweaver | 2020 v20.2.1 | :white_check_mark: | :x: | Works.
Dreamweaver | 2021 v21.1 | :white_check_mark: | :x: | Works.
Illustrator | 2021 v25.1 | :white_check_mark:| :x: | Works, no additional command lines.
Illustrator | 2021 v25.2.0 | :x:| :x: | Illustrator works well but the Zii 6.x patch (with Big Sur) is really screwing up ... the application crashes right away. The Zii 6.x patch needs updating.
Illustrator | 2021 v25.2.1 | :white_check_mark:| :x: | **No patch** Works, no additional command lines.
:new: Illustrator | 2021 v25.2.3 | :question: | :question: | ????
InCopy | 2021 v16.0.2 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
InCopy | 2021 v16.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
:new: InCopy | 2021 v16.2 | :question: | :question: | ????
InDesign | 2021 v16.0.2 | :white_check_mark: | :x: | Works, but "Open" doesn't work, no command lines. We need to do more research.
InDesign | 2021 v16.1 | :white_check_mark: | :x: | Works perfectly, no additional command lines.
:new: InDesign | 2021 v16.2.1 | :question: | :question: | ????
Lightroom Classic | v10.1 | :white_check_mark: | :x: | Works but you absolutely **Block Little Snitch or LuLu.**
Lightroom Classic | v10.1.1 |  :white_check_mark: | :x: | Works you absolutely **Block Little Snitch or LuLu.**
Lightroom Classic | v10.2 | :interrobang: | :x: | Works perfectly, but Camera Raw doesn't work.
Media Encoder | 2021 v15.0 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
Media Encoder | 2021 v15.1 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
:new: Media Encoder | 2021 v15.2 | :question: | :question: | ????
Photoshop | 2020 v21.2.3 | :white_check_mark: | :x: | Works.
Photoshop | 2020 v21.2.5 | :white_check_mark: | :x: | Works.
Photoshop | 2021 v22.0, v22.1, v22.1.1, v22.2, v22.3, **v22.3.1** | :interrobang: | :white_check_mark: | Works  ... but it crashes all the time. We need to do more research for a patch. **No patch** Intel CPU : Works a charm. AMD CPU : Launch properly but crash all times.
:new: Photoshop | 2021 v22.4 | :question: | :white_check_mark: | ????
Prelude | 2020 v9.0.3 | :white_check_mark: | :x: | Works.
Prelude | 2021 v10.0 | :white_check_mark: | :question: | **No patch** Works.
Premiere Pro | 2021 v15.0 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
Premiere Pro | 2021 v15.1 | :white_check_mark: | :x: | **No patch** Works perfectly, no additional command lines.
:new: Premiere Pro | 2021 v15.2 | :question: | :question: | ????
Premiere Rush | v1.5.50 | :white_check_mark: | :x: | **No patch** Works.
Premiere Rush | v1.5.54 | :white_check_mark: | :x: | **No patch** Works.
:new: Premiere Rush | v1.5.58 | :question: | :question: | ????
XD | v36.2.32 | :white_check_mark: | :x: | **Block Little Snitch or LuLu.** Works, no additional command lines.
XD | v38.0.12 | :white_check_mark: | :x: | **Block Little Snitch or LuLu.** Works, no additional command lines.
XD | v39.0.12 | :white_check_mark: | :x: | **Block Little Snitch or LuLu.** Works, no additional command lines.
:new: XD | v40.0.22 | :question: | :question: | ?????
> Maybe the others need to be well controlled to be sure. Comments should help me! Thank you !

### Notes
- If you re-install any Adobe app then you will need redo the STEP-4 again.
