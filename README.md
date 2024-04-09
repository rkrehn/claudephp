# About
This guide is designed to teach people how to use Claude AI using a basic "I want one response" in PHP. This code is used in my travel itinerary generation website [ReisPlan](https://www.reisplan.net) after I migrated it away from ChatGPT. I love how PHP handles JSON and XML data, so this tutorial is a bit easier. I'll be using curl for sending the data.

# Setup

Declare your API key (insert your [Claude AI API key](https://console.anthropic.com/dashboard) in the quotes)

```PHP
// API endpoint URL
$url = 'https://api.anthropic.com/v1/messages';

// Your OpenAI API key
$apiKey = '[API KEY]';
```

# Setting up the request

The following code creates the JSON request to Claude AI. You can review all the options you can include here: [https://docs.anthropic.com/claude/reference/messages_post](https://docs.anthropic.com/claude/reference/messages_post).

It's worth noting that only the "model", "messages", and "max_tokens" are required in the body. There are [many models](https://docs.anthropic.com/claude/docs/models-overview), but I'm using **claude-instant-1.2** in this example. Likewise, there are a few roles you can use such as system, user, assistance, or function. Typically, the **system** role defines what you want to accomplish. In our example, we'll tell Claude AI **sytem** role that is acting as a travel agent (used on my [ReisPlan](https://www.reisplan.net) site).

First, we prepare the prompt statement:

```PHP
  // Prompt for the conversation
    $messages = [
        [
            'role' => 'user',
            'content' => 'You are a travel advisor that will deliver a detailed itinerary based on the information provided by the user.'
        ]
    ];
```

Next, we build a JSON array. In the below scenario, I'm only sending the prompt and the model using an **array** and then encoding it into JSON data using the **json_encode** function.

```PHP
// Convert messages to JSON
  $data = [
      'model' => 'claude-instant-1.2',
      'max_tokens' => 2048,
      'messages' => $messages
  ];

$jsonData = json_encode($postData);
```

Next, we set up the headers using an **array** to send the request using the Authorization header with our API Key and JSON content type. The curl array will set up the headers using **post** and the content using the **$jsonData** we built above. 

```PHP
// Setup the curl data
  $curl = curl_init();

  curl_setopt_array($curl, [
      CURLOPT_URL => $url,
      CURLOPT_RETURNTRANSFER => true,
      CURLOPT_ENCODING => '',
      CURLOPT_MAXREDIRS => 10,
      CURLOPT_TIMEOUT => 0,
      CURLOPT_FOLLOWLOCATION => true,
      CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
      CURLOPT_CUSTOMREQUEST => 'POST',
      CURLOPT_POSTFIELDS => $jsonData,
      CURLOPT_HTTPHEADER => [
          'x-api-key: ' . $apiKey,
          'anthropic-version: 2023-06-01',
          'content-type: application/json'
      ],
  ]);
```

The **curl_exec** function will execute the above curl command

```PHP
$response = curl_exec($curl);
```

Finally, we'll close the curl execution

```PHP
// Execute curl
curl_close($curl);
```

# Receiving the response

We'll just the **json_decode** function to read the $response variable from OpenAI.

```PHP
// Decode the API response
$responseData = json_decode($response, true);
```
You will receive a JSON response like this in the **$data** variable (don't copy this):

```JSON
{
  "id": "string",
  "content": [
    {
      "text": "string"
    },
    {
      "id": "string",
      "name": "string",
      "input": {}
    }
  ],
  "model": "string",
  "stop_reason": "end_turn",
  "stop_sequence": "string",
  "usage": {
    "input_tokens": 0,
    "output_tokens": 0
  }
}
```

* content[0] is the first response
* text is the response you're looking for

Next, we'll extract the **$responseData** using the above and then explode individual lines in the response. The **$content** variable contains the whole response.

But, I used **$lines** to break the reply response so I could get multiple lines of data and have them displayed on the website.

```PHP
// Extract the assistant's reply
$content = $responseData['content'][0]['text'];

// Output the reply
// Assuming $reply contains the response from the API
$lines = explode("\n", $content);
```

The following loop examines each **$line** received and displays it on the website using the echo function.

```PHP
// Loop through the lines
foreach ($lines as $line) {
    // Process each line as needed
    if (preg_match('/^\*\*Day \d+:\*\*$/', $line)) {
        echo '<strong>' . $line . "</strong><br>";
    } else {
        echo $line . '<br>';
    }
}
```


# Everything together

Now, put it together in all its glory:

```PHP
$url = 'https://api.anthropic.com/v1/messages';

// Your OpenAI API key
$apiKey = '[API KEY]'; 

// Claude
$model = 'claude-instant-1.2';
$maxTokens = 2048;
$temperature = 0.6;    
$messages = [
    [
        'role' => 'system',
        'content' => 'You are a travel advisor that will deliver a detailed itinerary based on the information provided by the user during the '.$season.' season. I am traveling to Denver, CO.'
    ]
];

$data = [
    'model' => $model,
    'max_tokens' => $maxTokens,
    'temperature' => $temperature,
    'messages' => $messages
];

$jsonData = json_encode($data);

$curl = curl_init();

curl_setopt_array($curl, [
    CURLOPT_URL => $url,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => '',
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => 'POST',
    CURLOPT_POSTFIELDS => $jsonData,
    CURLOPT_HTTPHEADER => [
        'x-api-key: ' . $apiKey,
        'anthropic-version: 2023-06-01',
        'content-type: application/json'
    ],
]);

$response = curl_exec($curl);

curl_close($curl);

$responseData = json_decode($response, true);

$content = $responseData['content'][0]['text'];
```

> Sure, I can recommend you check out Livin the Dream brewery in Littleton and Denver Beer Co in Englewood.
