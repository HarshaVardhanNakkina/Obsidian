# Smart Ship Operations - Instructions to run the application

> **Note:** before running the applications, make sure all applications and connections are configured with the correct IP addresses, and data is being transmitted by the sensors.

## To run all applications with single click
- open `File Explorer` from windows menu
- go to `Desktop`
- locate `smart_ship_operations.bat` file & double click it
- if the application asks for log in details, try _username_: admin, _password_: 1234

## To configure the IP address
- open `Manage Network Adapter Settings` from windows menu
- select the `adapter` which is currently `connected` to the `router`
- go to `View additional properties`
- click on `Edit` located next to the text `IP assignment`
- change `Automatic (DHCP)` to `Manual` in the drop down
- after that, `enable IPv4`
- assign IP address `192.168.2.90`, Subnet mask `255.255.255.0` , and `save`
- `Disable and Enable` the adapter, if the settings didn’t take effect.

## To access FLIR camera font-end application
- Visit the IP address of the FLIR camera (probably, `http://192.168.2.77`)
- Login with credentials
- Go to PTZ from the sidebar

## For screen recording
- open `Snipping Tool` from windows menu
- select the `Video icon` in the snipping tool window
- click on `New`
- `grab the area` you want to capture, using the cursor
- click on `Start` to start the screen recording
- to stop the recording `click` on the `red square`
- to avoid large file sizes, `stop, save, and restart` the recording once in `every 2 or 3 hours`
- `save` the recording on `Desktop`, in a folder named `KOCHI-TRIALS-04APR2024`

## For Wire Shark recording
- open `Wire Shark` from windows menu
- from the list of adapters shown, select `Adapter for loop back traffic capture`
- click on the blue icon from the top left menu to `start` the recording
- to `stop` the recording `click` on the `red square`
- to avoid large file sizes, `stop, save, and restart` the recording once in `every 2 or 3 hours`
- `save` the recording on `Desktop`, in a folder named `KOCHI-TRIALS-04APR2024`

## To run the AI application separately
- open `Terminal` app from windows menu
- enter the command `cd ~/coding/boxMOT/AICE_v.1`
- enter the command `~/coding/py-playground/yolo-webcam/env/Scripts/Activate.ps1`
- enter the command `python track.py`

## To run the UI application separately
- open `Terminal` app from windows menu
- enter the command `cd ~/coding/fpm-ui-new`
- enter the command `npm run dev`
- in a new `Terminal`, enter the command `cd ~/coding`
- enter the command `java -jar ./fpm-core-service-0.0.1-SNAPSHOT.jar`
- in a new `Terminal`, enter the command `cd .\Downloads\tools\nginx-1.25.3\`
- enter the command `start .\nginx.exe`
- open chrome, and open the link `http://127.0.0.1:5173/fpm-home/?w=1920&h=1080` (replace the 640 and 480 with the actual width and height of the image respectively)
- if the application asks for log in details, try _username_: admin, _password_: 1234
- in a new tab, open the link `https://127.0.0.1:8080/fpm/seaObjects_v2` and accept the certificate if asked
- if you are connected to a network, `replace 127.0.0.1 with the IP configured for your system`

## To run FLIR reverse proxy
- in a new `Terminal`, enter the command `cd ./Downloads/tools/nginx-1.25.3/`
- enter the command `start ./nginx.exe`
