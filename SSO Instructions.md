## To run all applications with single click
- open `File Explorer` from windows menu
- go to `Desktop`
- locate `smart_ship_operations.bat` file & ***double click*** it
## To run the AI application separately  
- open `Terminal` app from windows menu
- enter the command `cd ~/coding/boxMOT/Det+MTT-1_02-11-23`
- enter the command `~/coding/py-playground/yolo-webcam/env/Scripts/Activate.ps1`
- enter the command `python track.py`
## To run the UI application separately  
- open `Terminal` app from windows menu
- enter the command `cd ~/coding/fpm-ui-new`
- enter the command `npm run dev`
- in a new `Terminal`, enter the command `cd ~/coding`
- enter the command `java -jar ./fpm-core-service-0.0.1-SNAPSHOT.jar`
- open chrome, and open the link http://127.0.0.1:5173/fpm-home/?w=640&h=480 (replace the `640` and `480` with the actual `width` and `height` of the image respectively)
- in a new tab, open the link https://127.0.0.1:8080/fpm/seaObjects_v2 and accept the certificate if asked
- if you are connected to a network, replace `127.0.0.1` with the IP set for your system
## For screen recording  
- open `Snipping Tool` from windows menu
- select the `Video icon` in the snipping tool window
- click on `New`
- `grab the area` you want to capture, using the cursor
- click on `Start` to start the screen recording
- to stop the recording `click` on the `red square`
- to avoid large file sizes, `stop, save, and restart` the recording once in `every 2 or 3 hours`
- `save` the recording on `Desktop`, in a folder named `trails_kochi_feb-2024`
## For Wire Shark recording  
- open `Wire Shark` from windows menu
- from the list of adapters shown, select `Adapter for loop back traffic capture`
- `click` on the `blue icon` from the top left menu to `start` the recording
- to `stop` the recording `click` on the `red square`
- to avoid large file sizes, `stop, save, and restart` the recording once in every `2 or 3 hours`
- `save` the recording on `Desktop`, in a folder named `trails_kochi_feb-2024`
## To set the IP address  
- open `Manage Network Adapter Settings` from windows menu
- select the `adapter` which is currently `connected` to the `router`
- go to `View additional properties`
- click on `Edit` located next to the text `IP assignment`
- change `Automatic (DHCP)` to `Manual` in the drop down
- after that, `enable IPv4`
- assign IP address `192.168.100.10`, Subnet mask `255.255.255.0`, and `save`
- `Disable and Enable` the adapter, if the settings didn’t take effect.