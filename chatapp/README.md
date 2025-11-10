Commands executed:

git clone https://github.com/ibm-developer-skills-network/bkrva-chatapp-with-voice-and-openai-outline.git
mv bkrva-chatapp-with-voice-and-openai-outline chatapp-with-voice-and-openai-outline
cd chatapp-with-voice-and-openai-outline

mkdir /home/project/chatapp-with-voice-and-openai-outline/certs/
cp /usr/local/share/ca-certificates/rootCA.crt /home/project/chatapp-with-voice-and-openai-outline/certs/

# These commands first build the application (running the commands in the Dockerfile) and tags (names) the built container as voice-chatapp-powered-by-openai, then runs it in the foreground on port 8000.
docker build . -t voice-chatapp-powered-by-openai
docker run -p 8000:8000 voice-chatapp-powered-by-openai


======================================================================
HTML, CSS, and Javascript
The index.html file is responsible for the layout and structure of the web interface. This file contains the code for incorporating external libraries such as JQuery, Bootstrap, and FontAwesome Icons, as well as the CSS (style.css) and Javascript code (script.js) that control the styling and interactivity of the interface.

The style.css file is responsible for customizing the visual appearance of the page's components. It also handles the loading animation using CSS keyframes. Keyframes are a way of defining the values of an animation at various points in time, allowing for a smooth transition between different styles and creating dynamic animations.

The script.js file is responsible for the page's interactivity and functionality. It contains the majority of the code and handles all the necessary functions such as switching between light and dark mode, sending messages, and displaying new messages on the screen. It even enables the users to record audio.
================================================

The server.py file consists of 3 functions which are defined as routes, and the code to start the server.

Wrap Toggled!
When a user tries to load the application, they initially send a request to go to the / endpoint. They will then trigger this index function above and execute the code above. Currently, the returned code from the function is a render function to show the index.html file which is the frontend interface.

The second and third routes are what will be used to process all requests and handle sending information between the applications.

Finally, the application is started with the app.run command to run on port 8080 and have the host be 0.0.0.0

===============================
worker.py -> speech_to_text

The function simply takes audio_binary as the only parameter and then sends it in the body of the HTTP request.

To make an HTTP Post request to Watson Speech-to-Text API, you need the following:

URL of the API: This is defined as api_url in your code and points to the Watson's Speech-to-Text service
Parameters: This is defined as params in your code. It's just a dictionary having one key-value pair i.e. 'model': 'en-US_Multimedia' which simply tells Watson that you want to use the US English model for processing your speech
Body of the request: this is defined as body and is equal to audio_binary since you are sending the audio data inside the body of your POST request.
You then use the requests library to send this HTTP request passing in the url, params, and data(body) to it and then use .json() to convert the API's response to json format which is very easy to parse and can be treated like a dictionary in Python.

=========================================================
worker.py -> openai_process_message

The function is really simple, thanks to the very easy to use openai library.

This is where you can give your personal assistant some personality. In this case you are telling the model to become a personal assistant by: Act like a personal assistant, and then giving it’s specific tasks its capable of doing: You can respond to questions, translate sentences, summarize news, and give recommendations.. By adding the original user message afterwards, it gives OpenAI more room to sound genuine. Feel free to change this according to what you require.

Then you call OpenAI's API by using openai.chat.completions.create function and pass in the following 3 parameters:

model: This is the OpenAI model we want to use for processing our prompt, in this case we are using their gpt-3.5-turbo model.
messages: The messages parameter is an array of objects used to define the conversation flow between the user and the AI. Each object represents a message with two key attributes: role (identifying the sender as either “system” for setup instructions or “user” for the actual user query) and content (the message text). The “system” role message instructs the AI on how to behave (for example, acting like a personal assistant), while the “user” role message contains the user's input. This structured approach helps tailor the AI's responses to be more relevant and personalized.
max_tokens: This is the maximum length of the response we are looking for. 30 tokens correspond to roughly 1-2 sentences. Right now, we are setting it to 4000.

The structure of the response is something like this:

{
  "choices": [
      {"message": {
          content: "The model\'s answer to our prompt",
          ...,
          ...,
      },
      ...,
      ...
    ]
}

Hence, you parse the openai_response to extract the answer to your prompt which is openai_response.choices[0].message.content and store it in a variable called response_text. Finally, you return the response_text.

=============================
worker.py -> text_to_speech

The function simply takes text and voice as the parameters. It adds voice as a parameter to the api_url if it's not empty or not default. It sends the text in the body of the HTTP request.

Similarly, as before, to make an HTTP Post request to Watson Text-to-Speech API, you need the following three elements:

URL of the API: This is defined as api_url in your code and points to Watson's Text to Speech service. This time you also append a voice parameter to the api_url if the user has sent a preferred voice in their request.
Headers: This is defined as headers in your code. It's just a dictionary having two key-value pairs. The first is 'Accept':'audio/wav' which tells Watson that we are sending an audio having wav format. The second one is ‘Content-Type’:’application/json’, which means that the format of the body would be JSON
Body of the request: This is defined as json_data and is a dictionary containing ‘text’:text key-value pair, this text will then be processed and converted to a speech.
We then use the requests library to send this HTTP request passing in the URL, headers, and json(body) to it and then use .json() to convert the API's response to json format so we can parse it.

The structure of the response is something like this:

{
  "response": {
        content: The Audio data for the processed text to speech
    }
  }
}

Therefore, we return response.content which contains the audio data received.

===================================
server.py -> speech-to-text

We start off by storing the request.data in a variable called audio_binary, as we are sending the binary data of audio in the body of the request from the front end. Then we use our previously defined function speech_to_text and pass in the audio_binary as a parameter to it. We store the return value in a new variable called text.

As our front end expects a JSON response, we create a json response by using the Flask's app.response_class function and passing in three arguments:

response: This is the actual data that we want to send in the body of our HTTP response. We will be using json.dumps function and will pass in a simple dictionary containing only one key-value pair -'text': text
status: This is the status code of the HTTP response; we will set it to 200 which essentially means the response is OK and that the request has succeeded.
mimetype: This is the format of our response which is more formally written as 'application/json' in HTTP request/response.
We then return the response.
=========================
Process message route

This function will basically accept a user's message in text form with their preferred voice. It will then use our previously defined helper functions to call the OpenAI's API to process this prompt and then finally convert that response to text using Watson's Text to Speech API and then return this data back to the user.

We will start by storing user's message in user_message by using request.json['userMessage']. Similarly, we will also store the user's preferred voice in voice by using request.json['voice'].

We will then use the helper function we defined earlier to process this user's message by calling openai_process_message(user_message) and storing the response in openai_response_text. We will then clean this response to remove any empty lines by using a simple one liner function in python, os.linesep.join([s for s in openai_response_text.splitlines() if s]).

Once we have this response cleaned, we will now use another helper function we defined earlier to convert it to speech. Therefore, we will call text_to_speech and pass in the two required parameters which are openai_response_text and voice. We will store the function's return value in a variable called openai_response_speech.

As the openai_response_speech is a type of audio data, we can't directly send this inside a json as it can only store textual data. Therefore, we will be using something called “base64 encoding”. In simple words, we can convert any type of binary data to a textual representation by encoding the data in base64 format. Hence, we will simply use base64.b64encode(openai_response_speech).decode('utf-8') and store the result back to openai_response_speech.

Now we have everything ready for our response so finally we will be using the same app.response_class function and send in the three parameters required. The status and mimetype will be exactly the same as we defined them in our previous speech_to_text_route. In the response, we will use json.dumps function as we did before and will pass in a dictionary as a parameter containing "openaiResponseText":openai_response_text and "openaiResponseSpeech":openai_response_speech.

We then return the response.

=================
Run the application:

docker build . -t voice-chatapp-powered-by-openai
docker run -p 8000:8000 voice-chatapp-powered-by-openai

open the UI -> https://emailappdb9b-8000.theiadockernext-1-labs-prod-theiak8s-4-tor01.proxy.cognitiveclass.ai/

Now that you've built an application using these Speech-to-Text and Text-to-Speech capabilities, if you wish to use IBM Watson Speech Libraries for Embed in your own applications you can use the following links to sign up for free trials.

Speech-to-Text -> https://www.ibm.com/account/reg/us-en/signup?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBMSkillsNetwork-GPXX0IWWEN&formid=urx-51754
Text-to-Speech -> https://www.ibm.com/account/reg/us-en/signup?utm_source=skills_network&utm_content=in_lab_content_link&utm_id=Lab-IBMSkillsNetwork-GPXX0IWWEN&formid=urx-51758