# A flow to automatically get TLE data from celestrak.com and use it with the satellites node   
   
Navigation: [home](README.md)

There are two parts to consider here.   
You should not hit the URL every time you want to track the satellite or move the rotator. To pull this off, we get the TLE and store it in a flow.context, then every time we want to move the rotator or get the satellite position data we read the TLE out of the flow.context.   
This example uses 25E Alphasat as its example. Just change the satellite number for your object of tracking choice.
    
With all that said, here is the flow:   
   
    [{"id":"ff478ce1.f0b78","type":"inject","z":"dac61f27.3a12b8","name":"24 hours","props":[{"p":"payload"}],"repeat":"86400","crontab":"","once":true,"onceDelay":0.1,"topic":"","payload":"","payloadType":"date","x":1420,"y":1140,"wires":[["7f94c7f5.78a3d8"]]},{"id":"7aaf15aa.0224dc","type":"debug","z":"dac61f27.3a12b8","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":2210,"y":1240,"wires":[]},{"id":"7f94c7f5.78a3d8","type":"http request","z":"dac61f27.3a12b8","name":"get 25E TLE","method":"GET","ret":"txt","paytoqs":"ignore","url":"https://celestrak.com/satcat/tle.php?CATNR=39215","tls":"","persist":false,"proxy":"","authType":"","x":1600,"y":1140,"wires":[["76626a71.02b424"]]},{"id":"30bd0a6b.dcf856","type":"tle","z":"dac61f27.3a12b8","satid":"","tle1":"","tle2":"","coordsys":"latlongdeg","name":"25E Alphasat","x":2020,"y":1240,"wires":[["7aaf15aa.0224dc"]]},{"id":"7ee3cc64.1568f4","type":"debug","z":"dac61f27.3a12b8","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1930,"y":1180,"wires":[]},{"id":"76626a71.02b424","type":"function","z":"dac61f27.3a12b8","name":"set","func":"// From the returned string, split on new line and build\n//an array of sat, tle1 and tle2\n//then store it in the 'flow' memory so we can use it every 15min to drive the dish.\n\nlist = msg.payload.split('\\n');\nflow.set(\"25list\", list);\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":1800,"y":1140,"wires":[["d7e3839d.1f54f"]]},{"id":"ca011185.37668","type":"function","z":"dac61f27.3a12b8","name":"get 25list","func":"// Get the TLE from the flow memory and feed it into the array and feed that into the sat node\n// (All this so we don't pound the celestrak website every 15 minutes needlessly)\n\nlist = flow.get(\"25list\");\nmsg.payload = list;\n\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":1580,"y":1240,"wires":[["4735b9e9.3dbc68","64941ac2.789ef4"]]},{"id":"d39902e0.20b55","type":"inject","z":"dac61f27.3a12b8","name":"15 minutes","props":[{"p":"payload"}],"repeat":"900","crontab":"","once":false,"onceDelay":0.1,"topic":"","payload":"","payloadType":"date","x":1430,"y":1240,"wires":[["ca011185.37668"]]},{"id":"d7e3839d.1f54f","type":"debug","z":"dac61f27.3a12b8","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1980,"y":1140,"wires":[]},{"id":"64941ac2.789ef4","type":"debug","z":"dac61f27.3a12b8","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1710,"y":1180,"wires":[]},{"id":"4735b9e9.3dbc68","type":"change","z":"dac61f27.3a12b8","name":"tweak the payload","rules":[{"t":"set","p":"satid","pt":"msg","to":"payload[0]","tot":"msg"},{"t":"set","p":"tle1","pt":"msg","to":"payload[1]","tot":"msg"},{"t":"set","p":"tle2","pt":"msg","to":"payload[2]","tot":"msg"},{"t":"set","p":"timestamp","pt":"msg","to":"","tot":"date"},{"t":"delete","p":"payload","pt":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":1770,"y":1240,"wires":[["30bd0a6b.dcf856","7ee3cc64.1568f4"]]}]