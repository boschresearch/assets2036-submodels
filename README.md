# The Assets2036 submodel repository

This repository serves as central model store for submodel specifications used 
by the lightweight implementation of asset administration shells in use at ARENA2036.


## Purpose of the project

The artifacts in this repository are not ready for production use. 
It has neither been developed nor tested for a specific use case. 
However, the license conditions of the applicable Open Source licenses allow 
you to adapt the software to your needs. Before using it in a safety relevant 
setting, make sure that the software fulfills your requirements and 
adjust it according to any applicable safety standards (e.g. ISO 26262).


## Usage

### Creation of new Submodels

Try to avoid creating new submodels, first take a look on what is already there!
Reuse is always the better choice :wink:
If you create a new submodel, try to focus on one single aspect.
It is better to have an asset implement a number of smaller submodels that can 
be reused than to have it implement one big submodel that is only usable in this
special context.
Submodels must follow the json schema specified in
[submodel.schema](submodel.schema). 
You can use https://jsonschemalint.com/ to verify your submodel.

### Use of Submdodels for communication via MQTT and JSON

Assets communicate with each other using submodel specifications. The following
convention for MQTT topics is used for all communication:

`<namespace>/<asset>/<submodel>/<submodel element>`

where
- `namespace` is a structuring element to distuingish different assets. 
- `asset` is the name/identifier of the asset that owns the data (it should be unique within the namespace).
- `submodel` is the submodel's name as specified within the submodel description
- `submodel element` is the name of the respective submodel element (see below for specifics)

The different submodel elements are then communicated as follows:

- Properties
 
  The payload for properties is the properties' value directly encoded using JSON

- Events
 
  Events additionally send a timestamp to denote the time the event occured.
  The complete payload for an event is therefore encoded as an json object:

  `{"timestamp": <timstamp as iso8691 string>, "params": <specified parameters as json object>}`
  

- Operations
 
  Since operations allow for bidirectional communication, two subtopics `REQ` and `RESP` are used. The caller sends an operation request to `REQ` with the payload

  `{"req_id":<unique id as string>, "params": <operation parameters as json object>}`
  
  The operation response is then sent via `RESP` as
  
  `{"req_id":<same req_id as the request as string>, "resp": <response of the operation as specified in the submodel>}`

#### Example 

Consider the following simple submodel

```javascript
{
  "name": "light",
  "revision": "0.0.1",
  "description": "describes a single switchable light source",
  "properties": {
    "light_on": {
      "type": "boolean",
      "description": "true if light is on"
    }
  },
  "events": {
    "light_switched": {
      "description": "triggered whenever the light is switched, parameter 'state' tells if on or off",
      "parameters": {
        "state": {
          "type": "boolean"
        }
      }
    }
  },
  "operations": {
    "switch_light": {
      "description": "switches light on if state is set to True, off if state is False, returns true if operation successful",
      "parameters": {
        "state": {
          "type": "boolean"
        }
      },
      "response": {
        "type": "boolean"
      }
    }
  }
}
```

An asset "lamp_1" located at the arena2036 would communicate as follows

- Send a property update

  | Topic | payload |
  | ------ | ------ |
  | `arena2036/lamp_1/light/light_on` | `true` |

- Emit an event

  | Topic | payload |
  | ------ | ------ |
  | `arena2036/lamp_1/light/light_switched` | `{"timestamp": "2020-03-12T07:20:34.837608", "params": {"state": true}}` |
  
- Receive and answer an operation request

  Incoming request (e.g. from other asset):
  
  | Topic | payload |
  | ------ | ------ |
  | `arena2036/lamp_1/light/switch_light/REQ` | `{"req_id": "266e8e766-4e6b-45a3-b1d8-3b574237e8b2", "params": {"state": false}}` |
  
  Outgoing answer from lamp_1:
  
  | Topic | payload |
  | ------ | ------ |
  | `arena2036/lamp_1/light/switch_light/RESP` | `{"red_id": "66e8e766-4e6b-45a3-b1d8-3b574237e8b2", "resp": true}` |
  

### Self description and discovery

An asset needs to announce which submodels it implements so that other assets 
know how to communicate with it. This is achieved by using the property `_meta` which is an implicit part of each submodel.
An asset needs to send it's _meta-properties at least once on startup to be discoverable by others.

The payload of this meta property holds information about the submodel specification and the origin of the asset data:

```javascript
{
    "submodel_url": <url to json-file holding the submodel specification>,
    "submodel_definition": <submodel specification as json object>,
    "source": <name of the actual asset that sends this data>
    }
}

```

#### Example 

Following the above example, the asset "lamp_1" would need to send:

| Topic | payload |
| ------ | ------ |
| `arena2036/lamp_1/light/_meta` | `{"submodel_url": "https://.../light.json", "submodel_definition": {"name": "light", ...}, "source": "lamp_1"}` |


## License

The submodels are open-sourced under the MIT license. See the [LICENSE](LICENSE) file for details.

## Quality assurance


## Known issues/limitations
