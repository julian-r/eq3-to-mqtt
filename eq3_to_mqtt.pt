import paho.mqtt.client as mqtt
from eq3bt import Thermostat
import json

THERMOSTATS = {
"00:1A:22:0E:A1:8E": "rooms/12.7/radiator1",
"00:1A:22:0E:A3:F4": "rooms/12.7/radiator2",
}


class ThermostatBridge(Thermostat):
def __init__(self, _mac, callback):
super().__init__(_mac)
self.callback = callback

def handle_notification(self, data):
super().handle_notification(data)

state = {
"firmware_version": self.firmware_version,
"device_serial": self.device_serial,
"target_temperature": self.target_temperature,
"locked": self.locked,
"boost": self.boost,
"low_battery": self.low_battery,
"valve_state": self.valve_state,
"window_open": self.window_open,
"away_end": self.away_end,
#    "window_open_time": self.window_open_time.seconds,
"eco_temperature": self.eco_temperature,
"temperature_offset": self.temperature_offset,
}

self.callback(self._conn.mac, state)


class BridgeServer:
def __init__(self, client):
self.connections = {}
self.mqtt_client = client
self.mqtt_client.on_connect = self.on_connect
self.mqtt_client.on_message = self.on_message

def on_thermostat_cb(self, mac, data):
topic = THERMOSTATS[mac]
print(data)
self.mqtt_client.publish(topic, json.dumps(data))

def on_connect(self, client, userdata, flags, rc):
# Subscribing in on_connect() means that if we lose the connection and
# reconnect then subscriptions will be renewed.
print("connected")
for mac, topic in THERMOSTATS.items():
print("mac")
client.subscribe("{}/set".format(topic))
self.connections[topic] = ThermostatBridge(mac, self.on_thermostat_cb)

# should be done async!
self.connections[topic].query_id()
self.connections[topic].update()

print("subscribed to all the things")

# The callback for when a PUBLISH message is received from the server.
def on_message(self, client, userdata, msg):
print("message!", msg.topic)
if not msg.topic.endswith("/set"):
return

connection = msg.topic[:-4]
print(connection)

request = json.loads(msg.payload)
print(request, connection)

# do stuff
if "target_temperature" in request:
self.connections[connection].target_temperature = request[
"target_temperature"
]


client = mqtt.Client()
server = BridgeServer(client)
client.connect("krake")

# Blocking call that processes network traffic, dispatches callbacks and
# handles reconnecting.
# Other loop*() functions are available that give a threaded interface and a
# manual interface.
client.loop_forever()
