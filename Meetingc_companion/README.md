# Preparing the environment
pip3 install virtualenv 
virtualenv my_env # create a virtual environment my_env
source my_env/bin/activate # activate my_env

# installing required libraries in my_env
pip install transformers==4.36.0 torch==2.1.1 gradio==5.23.2 langchain==0.0.343 ibm_watson_machine_learning==1.0.335 huggingface-hub==0.28.1

sudo apt update

# Learn more bout Gradio
If you wish to learn a little bit more about customization in Gradio, you are invited to take the guided project called Bring your Machine Learning model to life with Gradio. You can find it under Courses & Projects on https://cognitiveclass.ai/

# Script flow
1. speech2text_app.py -< speech 2 test functionality tested here>
2. simple_llm.py -> LLm functiinality tested here
3. speech_analyzer.py -> Final script contains all functinalities
    In this exercise, we'll set up a language model (LLM) instance, which could be IBM WatsonxLLM, HuggingFaceHub, or an OpenAI model. Then, we'll establish a prompt template. These templates are structured guides to generate prompts for language models, aiding in output organization (more info in langchain prompt template.

    Next, we'll develop a transcription function that employs the OpenAI Whisper model to convert speech-to-text. This function takes an audio file uploaded through a Gradio app interface (preferably in .mp3 format). The transcribed text is then fed into an LLMChain, which integrates the text with the prompt template and forwards it to the chosen LLM. The final output from the LLM is then displayed in the Gradio app's output textbox.