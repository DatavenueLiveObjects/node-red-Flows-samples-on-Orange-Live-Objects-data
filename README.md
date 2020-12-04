# node-red-orange-iot-samples 

## Presentation 

node-red-orange-iot-samples gives some explanations on default samples you will find while accessing Live Objects Node-RED option.

## Data transformation using Live Objects and Node-RED

Node-RED, the popular open source under Apache 2.0 license, has been integrated in SaaS mode within Live Objects, to allow users to describe and execute, through a flow graphical editor, their own data transformation treatments: format adaptation, decoder, data enrichment and more depending on users' needs and imagination.

See https://nodered.org/ to discover what Node-RED is.

To be able to use Node-RED features, ask your vendor for an access: you will receive a mail with your account and the URL to use Node-RED option.

Using this account, you can log-in the Node-RED SaaS page:

<img src="images/image1.png" width="350">

When opening the editor, you will find 3 treatments (called **flows** in Node-RED vocabulary) samples in the "Default samples" tab:

<img src="images/image1 bis.png" width="600">

You can either use this documentation, or follow embedded comments to do your first steps using Node-RED in link with Live Objects:
- *Getting data from a Live Objects MQTT topic (application mode)*, to base Node-RED treatment on data coming from Live Objects (through MQTT)
- *Sending data to a Live Objects MQTT topic (device mode)*, to send data transformed by Node-RED to Live Objects (through MQTT)
- *Sample 1: pushing to Live Objects data collected from a bike station*
- *Sample 2: enriching data, using Live Objects custom pipeline feature*
- *Sample 3: event processing on data coming from Live Objects*

See also https://nodered.org/docs/ to learn more about Node-RED use.

Note that in this SaaS implementation, Node-RED context is stored in database, to be persistent even in the case of a start/stop of the platform. Therefore, you have to use asynchronous mode (https://nodered.org/docs/user-guide/writing-functions#asynchronous-context-access) to access this context (here we used "change" nodes).

## Getting data from a Live Objects MQTT topic (application mode)

In Node-RED editor, use an mqtt out node, and position it on an existing Live Objects FIFO:

<img src="images/image2.png" width="471" height="325">

MQTT address and port are to be set:

<img src="images/image3.png" width="459" height="313">

With your API-KEY, in "Application" profile in Live Objects:

<img src="images/image4.png" width="421" height="213">

When saved, your node should appear connected :

<img src="images/image5.png" width="144" height="56">

You can start to create your Flow by importing (Ctrl-I) the following source (note that you will have to put your own API-KEY API-KEY in Password field of Node-RED screen above):

```
[{"id":"51e8b7e.dc73f48","type":"tab","label":"Flow 1","disabled":false,"info":""},{"id":"3313c1a9.1b0bee","type":"mqtt in","z":"51e8b7e.dc73f48","name":"","topic":"fifo/myFifo","qos":"2","datatype":"auto","broker":"f06f3a34.82ea7","x":100,"y":80,"wires":[[]]},{"id":"f06f3a34.82ea7","type":"mqtt-broker","z":"","name":"Live Objects","broker":"liveobjects.orange-business.com","port":"1883","clientid":"myapp_id","usetls":false,"compatmode":true,"keepalive":"30","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","closeTopic":"","closeQos":"0","closePayload":"","willTopic":"","willQos":"0","willPayload":""}]
```

## Sending data to a Live Objects MQTT topic (device mode)

In Node-RED editor, use an mqtt out node, and position it on an existing Live Objects FIFO:

<img src="images/image6.png" width="604" height="201">

Edit the different property (connection and security) of your MQTT topic, giving the device ID which will appear in Live Objects (here urn:lo:nsid:mqttnodered:my_device):

<img src="images/image7.png" width="445" height="344">

<img src="images/image8.png" width="456" height="235">

When saved, your node should appear connected :

<img src="images/image9.png" width="206" height="62">

You can start to create your Flow by importing (Ctrl-I) the following source (note that you will have to put your own device API-KEY in Password field of Node-RED screen above):

```
[{"id":"d8f9c0f5.9e7fc8","type":"mqtt out","z":"12ea5f3e.fcefb9","name":"publish to dev/data","topic":"dev/data","qos":"0","retain":"","broker":"f06f3a34.82ea7","x":110,"y":80,"wires":[]},{"id":"f06f3a34.82ea7","type":"mqtt-broker","z":"","name":"Live Objects Device","broker":"liveobjects.orange-business.com","port":"8883","tls":"e7258cda.0b59a8","clientid":"urn:lo:nsid:mqttnodered:my_device","usetls":true,"compatmode":true,"keepalive":"30","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","closeTopic":"","closeQos":"0","closePayload":"","willTopic":"","willQos":"0","willPayload":""},{"id":"e7258cda.0b59a8","type":"tls-config","z":"","name":"","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false}]
```

## Calling by HTTP a Node-RED treatment, at custom pipeline level

*Creation of the custom pipeline*

The custom pipeline mechanism of Live Objects is detailled in Live Objects documentation https://liveobjects.orange-business.com/doc/html/lo_manual_v2.html#PIPELINES). 

You first need to create in Live Objects, the custom pipeline, specifying the condition and (Node-RED treatment) URL you will call in this custom pipeline.

<img src="images/Image100.png" width="604" >

The creation of the custom pipeline can be done by Live Objects portal, like above, but also by API : you can use Node-RED to create it (one shot call).

<img src="images/image10.png" width="604" height="88">

Initialization of the custom pipeline is described in the JSON of the "inject" node.

The HTTP call of the Node-RED treatment will require an authentication. The basic key is issued from the base-64 coding of the chain <your_login>:<your_password> (for instance using https://www.base64encode.org/).

<img src="images/image11.png" width="526" height="415">
  
For instance:

```
{
    "enabled": true,
    "name": "test pipeline",
    "priorityLevel": 0,
    "steps": [
        {
            "type": "externalTransformation",
            "name": "appspot data enricher",
            "url": "https://node-red-1-nodered-enabler-prod.eu-b.kmt.orange.com/node-red/api/5e8dc6b44311625edbefa554/transform",
            "headers": {
                "Authorization": [
                    "Basic XXXXXXXXXXXXXXXXXXXXXXXX"
                ]
            }
        }
    ]
}

```

Note that this URL may differs in your case. Use the one received in the welcome mail.

Preparation of the call of Live Objects API to create the pipeline is done in "function" node:

<img src="images/image12.png" width="451" height="323">

And then, you can call the API with "http request" node:

<img src="images/image13.png" width="453" height="451">

*Treatment to be called by custom pipeline*

You can then define a flow that will be called when Live Objects receives messages, depending on the conditions you have set in the definition of your custom pipeline.

In the example below, we use the treatment called by custom pipeline to add in the stream of data that will be stored by Live Objects, external temperature value, get from an Internet API.

<img src="images/image14.png" width="604" height="137">

To fit with the definition of the custom pipeline done above, you first have to use a "http in" node, and name it accordingly (here "transform").

<img src="images/image15.png" width="360" height="219">

Then you can add the temperature value (here converted from Kelvin to Degree) in the payload that will be returned to Live Objects by the "http response" node:

<img src="images/image16.png" width="394" height="351">

You can start to create your Flow by importing (Ctrl-I) the following source (note that you will have to put your own API-KEYs):

```
[{"id":"91b958b6.9af578","type":"tab","label":"[e] Pipeline","disabled":false,"info":""},{"id":"eb2c2a85.5abf8","type":"http in","z":"91b958b6.9af578","name":"transform","url":"/transform","method":"post","upload":false,"swaggerDoc":"","x":100,"y":180,"wires":[["28317e7.4a6b082"]]},{"id":"afa96237.91ba2","type":"http response","z":"91b958b6.9af578","name":"","statusCode":"","headers":{},"x":650,"y":180,"wires":[]},{"id":"28317e7.4a6b082","type":"function","z":"91b958b6.9af578","name":"store initial msg","func":"msg.request = msg.payload;\nmsg.nom_arrondissement_communes = msg.payload.value.nom_arrondissement_communes;\n//msg.lat = msg.payload.location.lat;\n//msg.lon = msg.payload.location.lon;\nreturn msg;","outputs":1,"noerr":0,"x":180,"y":240,"wires":[["b96bc50a.fcd9d"]]},{"id":"55fabfa.e9ec64","type":"http request","z":"91b958b6.9af578","name":"","method":"use","ret":"txt","paytoqs":false,"url":"","tls":"","persist":false,"proxy":"","authType":"","x":410,"y":80,"wires":[["969f4c73.a2091"]]},{"id":"efd15f88.c548f8","type":"inject","z":"91b958b6.9af578","name":"","topic":"","payload":"{\"enabled\":true,\"name\":\"test pipeline\",\"priorityLevel\":0,\"steps\":[{\"type\":\"externalTransformation\",\"name\":\"appspot data enricher\",\"url\":\"https://node-red-1-nodered-enabler-prod.eu-b.kmt.orange.com/node-red/api/5e8dc6b44311625edbefa554/transform\",\"headers\":{\"Authorization\":[\"Basic ===APIKEY===\"]}}]}","payloadType":"json","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":110,"y":80,"wires":[["dbb55649.2d4a1"]]},{"id":"dbb55649.2d4a1","type":"function","z":"91b958b6.9af578","name":"set URL","func":"msg.headers = {};\nmsg.headers['Content-Type'] = 'application/json';\nmsg.headers['Accept'] = 'application/json';\nmsg.headers['X-API-KEY'] = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';\n\nmsg.method = \"POST\";\n\nmsg.url = \"https://liveobjects.orange-business.com/api/v0/pipelines\";\n\nreturn msg;","outputs":1,"noerr":0,"x":260,"y":80,"wires":[["55fabfa.e9ec64"]]},{"id":"969f4c73.a2091","type":"debug","z":"91b958b6.9af578","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":590,"y":80,"wires":[]},{"id":"ca3a29f4.b50708","type":"comment","z":"91b958b6.9af578","name":"Pipeline creation (on shot call)","info":"","x":160,"y":40,"wires":[]},{"id":"425ec7ff.ea76d8","type":"comment","z":"91b958b6.9af578","name":"Transformation called by the custom pipeline to enrich device data (here {{city}} T°)","info":"","x":330,"y":140,"wires":[]},{"id":"b96bc50a.fcd9d","type":"http request","z":"91b958b6.9af578","name":"","method":"GET","ret":"obj","paytoqs":false,"url":"http://api.openweathermap.org/data/2.5/weather?q={{nom_arrondissement_communes}},fr&appid=XXXXXXX","tls":"","persist":false,"proxy":"","authType":"","x":370,"y":240,"wires":[["d63aa8d5.e2319"]]},{"id":"d63aa8d5.e2319","type":"function","z":"91b958b6.9af578","name":"Add temperature","func":"msg.tmp = msg.payload;\nmsg.payload = msg.request;\n\nmsg.payload.value.temperature = msg.tmp.main.temp - 273.15;\n\nreturn msg;","outputs":1,"noerr":0,"x":550,"y":240,"wires":[["afa96237.91ba2"]]}]
```

Note that this URL may differs in your case. Use the one received in the welcome mail.

## Complete sample 2: data from 2 bike stations

This sample get every 30 minutes information from 2 bike stations in Paris (velib), and push them, as message from devices, to Live Objects:

<img src="images/image17.png" width="604" height="388">

You can start to create your Flow by importing (Ctrl-I) the following source (note that you will have to put your own API-KEY):

```
[{"id":"e27fb494.cbb5d8","type":"tab","label":"[c] Velib stations","disabled":false,"info":""},{"id":"80424dd0.46ee4","type":"http request","z":"e27fb494.cbb5d8","name":"GET pernety velib station last status","method":"GET","ret":"obj","paytoqs":false,"url":"https://opendata.paris.fr/api/records/1.0/search/?dataset=velib-disponibilite-en-temps-reel&q=pernety&rows=1","tls":"","persist":false,"proxy":"","authType":"","x":310,"y":120,"wires":[["80b868e7.1c43c8"]]},{"id":"17f1d44c.aeeee4","type":"inject","z":"e27fb494.cbb5d8","name":"Send Data Message trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"*/30 0-23 * * *","once":true,"onceDelay":"1","x":200,"y":60,"wires":[["80424dd0.46ee4"]]},{"id":"80b868e7.1c43c8","type":"template","z":"e27fb494.cbb5d8","name":"convert to Data Message","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{\n   \"ts\":  \"{{payload.records.0.record_timestamp}}\",\n   \"m\":   \"velibDispo1\",\n   \"loc\": [{{payload.records.0.geometry.coordinates.1}}, {{payload.records.0.geometry.coordinates.0}}],\n   \"v\": {\n      \"ebike\": {{payload.records.0.fields.ebike}},\n      \"capacity\": {{payload.records.0.fields.capacity}},\n      \"name\": \"{{payload.records.0.fields.name}}\",\n      \"nom_arrondissement_communes\": \"{{payload.records.0.fields.nom_arrondissement_communes}}\",\n      \"numbikesavailable\": {{payload.records.0.fields.numbikesavailable}},\n      \"mechanical\": {{payload.records.0.fields.mechanical}}\n   },\n   \"t\" : [ \"velib-disponibilite-en-temps-reel\"]\n}","output":"json","x":330,"y":180,"wires":[["451c1d6d.8d4854"]]},{"id":"451c1d6d.8d4854","type":"mqtt out","z":"e27fb494.cbb5d8","name":"publish to dev/data","topic":"dev/data","qos":"0","retain":"","broker":"8d30a774.31859","x":550,"y":180,"wires":[]},{"id":"1c0fc18.77cb93f","type":"mqtt out","z":"e27fb494.cbb5d8","name":"publish to dev/data","topic":"dev/data","qos":"0","retain":"","broker":"aeb99171.4817d","x":550,"y":360,"wires":[]},{"id":"142570d0.f59fc7","type":"template","z":"e27fb494.cbb5d8","name":"convert to Data Message","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{\n   \"ts\":  \"{{payload.records.0.record_timestamp}}\",\n   \"m\":   \"velibDispo1\",\n   \"loc\": [{{payload.records.0.geometry.coordinates.1}}, {{payload.records.0.geometry.coordinates.0}}],\n   \"v\": {\n      \"ebike\": {{payload.records.0.fields.ebike}},\n      \"capacity\": {{payload.records.0.fields.capacity}},\n      \"name\": \"{{payload.records.0.fields.name}}\",\n      \"nom_arrondissement_communes\": \"{{payload.records.0.fields.nom_arrondissement_communes}}\",\n      \"numbikesavailable\": {{payload.records.0.fields.numbikesavailable}},\n      \"mechanical\": {{payload.records.0.fields.mechanical}}\n   },\n   \"t\" : [ \"velib-disponibilite-en-temps-reel\"]\n}","output":"json","x":330,"y":360,"wires":[["1c0fc18.77cb93f"]]},{"id":"ef97f6f.6003c88","type":"http request","z":"e27fb494.cbb5d8","name":"GET marne velib station last status","method":"GET","ret":"obj","paytoqs":false,"url":"https://opendata.paris.fr/api/records/1.0/search/?dataset=velib-disponibilite-en-temps-reel&q=\"Marne - Charles de Gaulle\"&rows=1","tls":"53f20f00.72b838","persist":false,"proxy":"","authType":"","x":300,"y":300,"wires":[["142570d0.f59fc7"]]},{"id":"a2878a87.b8fa88","type":"inject","z":"e27fb494.cbb5d8","name":"Send Data Message trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"*/30 0-23 * * *","once":true,"onceDelay":"1","x":200,"y":240,"wires":[["ef97f6f.6003c88"]]},{"id":"9e1c4.30cf3e3d","type":"comment","z":"e27fb494.cbb5d8","name":"Push to Live Objects data from 2 velib stations, seen as 2 MQTT devices","info":"","x":320,"y":20,"wires":[]},{"id":"8d30a774.31859","type":"mqtt-broker","z":"","name":"Live Objects pernety","broker":"liveobjects.orange-business.com","port":"8883","tls":"e7258cda.0b59a8","clientid":"urn:lo:nsid:mqttnodered:pernety","usetls":true,"compatmode":false,"keepalive":"30","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","closeTopic":"","closeQos":"0","closePayload":"","willTopic":"","willQos":"0","willPayload":""},{"id":"aeb99171.4817d","type":"mqtt-broker","z":"","name":"Live Objects alfortville","broker":"liveobjects.orange-business.com","port":"8883","tls":"81c1449a.3fbf38","clientid":"urn:lo:nsid:mqttnodered:alfortville","usetls":true,"compatmode":false,"keepalive":"30","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","closeTopic":"","closeQos":"0","closePayload":"","willTopic":"","willQos":"0","willPayload":""},{"id":"53f20f00.72b838","type":"tls-config","z":"","name":"TLS-config-basic-no-checkings","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false},{"id":"e7258cda.0b59a8","type":"tls-config","z":"","name":"","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false},{"id":"81c1449a.3fbf38","type":"tls-config","z":"","name":"tls-config-verifyServerCertif","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"DigiCertRootCaCertif.pem","servername":"","verifyservercert":true}]
```


## Complete sample 3: event processing based on data from 2 devices

This sample retrieves data from Live Objects (North bound interface), to execute an event processing treatment:

* Based on data coming from several devices
* Using persistent context (*)

Especially with these features, Node-RED is complementary to what is provided directly by Live Objects native Event Processing mechanisms.

(*): Note that Node-RED context is stored in database, to be persistent even in the case of a start/stop of the platform. Therefore, you have to use asynchronous mode (https://nodered.org/docs/user-guide/writing-functions#asynchronous-context-access) to access this context (here we used "change" nodes).

<img src="images/image18.png" width="604" height="256">

You can start to create your Flow by importing (Ctrl-I) the following source (note that you will have to put your own API-KEY):

```
[{"id":"7ef560d1.b5be08","type":"tab","label":"[p] Velib alert","disabled":false,"info":""},{"id":"afe49b9c.d12818","type":"mqtt in","z":"7ef560d1.b5be08","name":"","topic":"fifo/vantiqFifo","qos":"2","datatype":"auto","broker":"f06f3a34.82ea7","x":90,"y":160,"wires":[["395a761c.b8da4a"]]},{"id":"395a761c.b8da4a","type":"json","z":"7ef560d1.b5be08","name":"","property":"payload","action":"obj","pretty":false,"x":230,"y":160,"wires":[["978ebb43.458298"]]},{"id":"8c1a9d4f.e0932","type":"debug","z":"7ef560d1.b5be08","name":"","active":true,"tosidebar":true,"console":false,"tostatus":true,"complete":"total","targetType":"msg","x":380,"y":360,"wires":[]},{"id":"3a785286.5be31e","type":"function","z":"7ef560d1.b5be08","name":"sum availabilities","func":"// get values from msg\nvar numbikes = msg.payload.value.numbikesavailable;\nvar name = msg.payload.value.name;\n\n// update availability array\nif (msg.availability === null ) msg.availability={};\nvar availability = msg.availability;\navailability[name]=numbikes;\n\n// calculate total availability\nvar total = 0;\nObject.keys(availability).forEach(k=>{node.warn(k+' => '+availability[k]); total += availability[k]})\n//node.warn(\"total : \" + total);\n\nmsg.payload = msg.payload + \"|\" + availability[\"Pernety - Raymond Losserand\"] + \"|\" + availability[\"Marne - Charles de Gaulle\"] + \"|\";\nmsg.total = total;\n\n//msg.topic   = name; // mandatory for charts to display a label\n\nreturn msg;","outputs":1,"noerr":0,"x":330,"y":300,"wires":[["8c1a9d4f.e0932","f9371a6e.33397","68e60697.17fa88"]]},{"id":"f9371a6e.33397","type":"change","z":"7ef560d1.b5be08","name":"set context","rules":[{"t":"set","p":"availability","pt":"global","to":"availability","tot":"msg"}],"action":"","property":"","from":"","to":"","reg":false,"x":530,"y":240,"wires":[[]]},{"id":"978ebb43.458298","type":"change","z":"7ef560d1.b5be08","name":"get context","rules":[{"t":"set","p":"availability","pt":"msg","to":"availability","tot":"global"}],"action":"","property":"","from":"","to":"","reg":false,"x":230,"y":240,"wires":[["3a785286.5be31e"]]},{"id":"7189df10.cb8a9","type":"inject","z":"7ef560d1.b5be08","name":"","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":100,"y":60,"wires":[["44087f0b.d3522"]]},{"id":"44087f0b.d3522","type":"change","z":"7ef560d1.b5be08","name":"","rules":[{"t":"delete","p":"availability","pt":"global"}],"action":"","property":"","from":"","to":"","reg":false,"x":290,"y":60,"wires":[[]]},{"id":"7a12ba0d.dbbd24","type":"inject","z":"7ef560d1.b5be08","name":"For test","topic":"","payload":"{\"value\":{\"name\":\"Lans-en-Vercors\",\"numbikesavailable\":5}}","payloadType":"json","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":70,"y":360,"wires":[["978ebb43.458298"]]},{"id":"68e60697.17fa88","type":"switch","z":"7ef560d1.b5be08","name":"if > 100","property":"total","propertyType":"msg","rules":[{"t":"gt","v":"100","vt":"num"},{"t":"lte","v":"100","vt":"num"}],"checkall":"true","repair":false,"outputs":2,"x":520,"y":300,"wires":[["bf5ea7.9061b158","bdf99d96.aef4f8"],["474365d6.a0c29c"]]},{"id":"bf5ea7.9061b158","type":"debug","z":"7ef560d1.b5be08","name":"ALERT ","active":true,"tosidebar":true,"console":false,"tostatus":true,"complete":"\"ALERT\"","targetType":"jsonata","x":690,"y":340,"wires":[]},{"id":"474365d6.a0c29c","type":"debug","z":"7ef560d1.b5be08","name":"DO NOTHING","active":true,"tosidebar":true,"console":false,"tostatus":true,"complete":"\"DO NOTHING\"","targetType":"jsonata","x":720,"y":420,"wires":[]},{"id":"d56de42b.48f7c","type":"http request","z":"7ef560d1.b5be08","name":"","method":"use","ret":"txt","paytoqs":false,"url":"","tls":"","persist":false,"proxy":"","authType":"","x":1010,"y":300,"wires":[[]]},{"id":"bdf99d96.aef4f8","type":"template","z":"7ef560d1.b5be08","name":"set mail","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"{\n \"contact\": \n  {\n    \"to\": [\n      \"jeanmichel.ortholand@orange.com\"\n    ]\n  },\n  \"message\": {\n     \"title\": \"Hello\",\n     \"content\": \"Hello World !\"\n  }\n}\n","output":"json","x":700,"y":300,"wires":[["99cc3663.80423"]]},{"id":"99cc3663.80423","type":"function","z":"7ef560d1.b5be08","name":"set request","func":"msg.headers = {};\nmsg.headers['Content-Type'] = 'application/json';\nmsg.headers['Accept'] = 'application/json';\nmsg.headers['X-API-KEY'] = '==APIKEY==';\n\nmsg.method = \"POST\";\n\nmsg.url = \"https://liveobjects.orange-business.com/api/v0/contact\";\n\nreturn msg;","outputs":1,"noerr":0,"x":850,"y":300,"wires":[["d56de42b.48f7c"]]},{"id":"18bc76ce.073371","type":"comment","z":"7ef560d1.b5be08","name":"This first flow allow to reset the context","info":"","x":170,"y":20,"wires":[]},{"id":"b9afd010.07d958","type":"comment","z":"7ef560d1.b5be08","name":"Get from Live Objects velib stations availabilities to sum and alert if sum > 100","info":"","x":290,"y":120,"wires":[]},{"id":"7207641a.4fc44c","type":"comment","z":"7ef560d1.b5be08","name":"Dummy station to test","info":"","x":120,"y":320,"wires":[]},{"id":"d32d73e.227a41","type":"comment","z":"7ef560d1.b5be08","name":"Format and send a mail with Live Objects if sum > 100","info":"","x":840,"y":260,"wires":[]},{"id":"f06f3a34.82ea7","type":"mqtt-broker","z":"","name":"Live Objects Device","broker":"liveobjects.orange-business.com","port":"8883","tls":"e7258cda.0b59a8","clientid":"urn:lo:nsid:mqttnodered:my_device","usetls":true,"compatmode":true,"keepalive":"30","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","closeTopic":"","closeQos":"0","closePayload":"","willTopic":"","willQos":"0","willPayload":""},{"id":"e7258cda.0b59a8","type":"tls-config","z":"","name":"","cert":"","key":"","ca":"","certname":"","keyname":"","caname":"","servername":"","verifyservercert":false}]
```

