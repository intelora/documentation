[Rest API]

**-------**

Intelora provides the REST API to manage the Synapses. 
For configuring the API, the user can refer to the settings documentation -> (settings.md).

[Synapse API]

| Method | URL                               | Action                             |
|--------|-----------------------------------|------------------------------------|
| GET    | /                                 | Get intelora version                |
| GET    | /synapses                         | List synapses                      |
| GET    | /synapses/<synapse_name>          | Get synapse details by name        |
| POST   | /synapses/start/id/<synapse_name> | Run a synapse by its name          |
| POST   | /synapses/start/order             | Run a synapse from a text order    |
| POST   | /synapses/start/audio             | Run a synapse from an audio sample |
| GET    | /mute                             | Get the current mute status        |
| POST   | /mute                             | Switch the mute status             |

**Curl examples**

[NOTE] --user is only needed if `password_protected` is True

[Get Intelora's version]

Normal response codes: 200
Error response codes: unauthorized(401)
Curl command:
```bash
curl -i --user admin:secret -X GET  http://localhost:5000/
```
Output example:
```JSON
{
  "Intelora version": "0.4.2"
}
```

[List synapses]

Normal response codes: 200
Error response codes: unauthorized(401), itemNotFound(404)
Curl command:
```bash
curl -i --user admin:secret -X GET  http://localhost:5000/synapses
```

Output example:

```JSON
{
  "synapses": [
    [
      {
        "name": "stop-intelora",
        "neurons": [
          {
            "say": {
              "message": "Good bye"
            }
          },
          "kill_switch"
        ],
        "signals": [
          {
            "order": "close"
          }
        ]
      }
    ],
    [
      {
        "name": "say-hello",
        "neurons": [
          {
            "say": {
              "message": [
                "Bonjour monsieur"
              ]
            }
          }
        ],
        "signals": [
          {
            "order": "bonjour"
          }
        ]
      }
    ]
  ]
}
```

**Show synapse details** 

Normal response codes: 200
Error response codes: unauthorized(401), itemNotFound(404)
Curl command:
```bash
curl -i --user admin:secret -X GET  http://localhost:5000/synapses/say-hello
```

Output example:
```JSON
{
  "synapses": {
    "name": "say-hello",
    "neurons": [
      {
        "say": {
          "message": [
            "Bonjour monsieur"
          ]
        }
      }
    ],
    "signals": [
      {
        "order": "bonjour"
      }
    ]
  }
}
```

**Run a synapse by its name**

Normal response codes: 201
Error response codes: unauthorized(401), itemNotFound(404)
Curl command:
```bash
curl -i --user admin:secret -X POST  http://localhost:5000/synapses/start/id/say-hello
```

Output example:

```JSON
{
  "matched_synapses": [
    {
      "matched_order": null,
      "neuron_module_list": [
        {
          "generated_message": "Bonjour monsieur",
          "neuron_name": "Say"
        }
      ],
      "synapse_name": "say-hello-fr"
    }
  ],
  "status": "complete",
  "user_order": null
}
```

The [no_voice flag](#no-voice-flag) can be added to this call.
Curl command:
```bash
curl -i -H "Content-Type: application/json" --user admin:secret -X POST \
-d '{"no_voice":"true"}' http://127.0.0.1:5000/synapses/start/id/say-hello-fr
```

There are some neuron inside a synapse that will wait for parameters that comes from the order. 
The user can provide those parameters by adding a `parameters` list of data.

Curl command:
```bash
curl -i -H "Content-Type: application/json" --user admin:secret -X POST  \
-d '{"parameters": {"parameter1": "value1" }}' \
http://127.0.0.1:5000/synapses/start/id/synapse-id
```

**Run a synapse from an order**

Normal response codes: 201
Error response codes: unauthorized(401), itemNotFound(404)

Curl command:

```bash
curl -i --user admin:secret -H "Content-Type: application/json" -X POST -d '{"order":"my order"}' http://localhost:5000/synapses/start/order
```

If the order contains accent or quotes, use a file for testing with curl

```bash
cat post.json 
{"order":"j'aime"}
```
Then,

```bash
curl -i --user admin:secret -H "Content-Type: application/json" -X POST --data @post.json http://localhost:5000/synapses/start/order
```

Output example if the order have matched and so launched synapses:

```JSON
{
  "matched_synapses": [
    {
      "matched_order": "Bonjour",
      "neuron_module_list": [
        {
          "generated_message": "Bonjour monsieur",
          "neuron_name": "Say"
        }
      ],
      "synapse_name": "say-hello-fr"
    }
  ],
  "status": "complete",
  "user_order": "bonjour"
}
```

If the order haven't match any synapses, it will try to execute the default synapse if it exists in the settings:

```JSON
{
  "matched_synapses": [
    {
      "matched_order": null,
      "neuron_module_list": [
        {
          "generated_message": "Je n'ai pas compris votre ordre",
          "neuron_name": "Say"
        }
      ],
      "synapse_name": "default-synapse"
    }
  ],
  "status": "complete",
  "user_order": "not existing order"
}
```

Or return an empty list of matched synapse

```
{
  "matched_synapses": [],
  "status": null,
  "user_order": "not existing order"
}
```

The [no_voice flag](#no-voice-flag) can be added to this call.

Curl command:

```bash
curl -i --user admin:secret -H "Content-Type: application/json" -X POST \
-d '{"order":"my order", "no_voice":"true"}' http://localhost:5000/synapses/start/order
```

**Run a synapse from an audio file**

Normal response codes: 201
Error response codes: unauthorized(401), itemNotFound(404)

The audio file must contain WAV or MP3 extension.

Curl command:

```bash
curl -i --user admin:secret -X POST  http://localhost:5000/synapses/start/audio -F "file=@/home/nico/Desktop/input.wav"
```

Output example if the order inside the audio have matched and so launched synapses:

```JSON
{
  "matched_synapses": [
    {
      "matched_order": "Bonjour",
      "neuron_module_list": [
        {
          "generated_message": "Bonjour monsieur",
          "neuron_name": "Say"
        }
      ],
      "synapse_name": "say-hello-fr"
    }
  ],
  "status": "complete",
  "user_order": "bonjour"
}
```

If the order haven't match any synapses it will try to execute the default synapse if it exists in the settings:

```JSON

{
  "matched_synapses": [
    {
      "matched_order": null,
      "neuron_module_list": [
        {
          "generated_message": "Je n'ai pas compris votre ordre",
          "neuron_name": "Say"
        }
      ],
      "synapse_name": "default-synapse"
    }
  ],
  "status": "complete",
  "user_order": "not existing order"
}
```

Or return an empty list of matched synapse

```
{
  "matched_synapses": [],
  "status": null,
  "user_order": "not existing order"
}
```

The [no_voice flag](#no-voice-flag) can be added to this call with a form.

Curl command:

```bash
curl -i --user admin:secret -X POST http://localhost:5000/synapses/start/audio -F "file=@path/to/file.wav" -F no_voice="true"
```

**Get mute status**

Normal response codes: 200
Error response codes: unauthorized(401), Bad request(400)

Curl command:

```bash
curl -i --user admin:secret  -X GET  http://127.0.0.1:5000/mute
```

Output example:

```JSON
{
  "mute": true
}
```

**Switch mute status**

Normal response codes: 200
Error response codes: unauthorized(401), Bad request(400)

Curl command:

```bash
curl -i -H "Content-Type: application/json" --user admin:secret  -X POST -d '{"mute": "True"}' http://127.0.0.1:5000/mute
```

Output example:

```JSON
{
  "mute": true
}
```

[No voice flag]

When the API is used, by default, Intelora will generate a text and process it into the TTS engine.
Some calls to the API can be done with a flag that will tell Intelora to only return the generated text without processing it into the audio player.

When `no_voice` is switched to true, Intelora will not speak out loud on the server side.
