## Incorporating AI in Your Applications

Getting Started

1. Understand the AI Landscape.  There are different kinds of AI and content-handling
   - [] **Text-based Models**: chatbots, translators, & summarizers. OpenAI GPT, Calude, Mistral
   - [] **Image-based Models** for recognition & generation. OpenAI DALL-E, Stability AI, OpenCV + ML
   - [] **Speech** to text or from text. Whisper, Amazon Transcribe, Azure Speech
   - [] **Recommendation** custom models and APIs
   - [] **Prediction**

2. Get Familiar with AI APIs    
   Choose a provider and learn how to
   - [] Make API calls
   - [] Pass structured prompts and data
   - [] Parse model responses
   Example: with OpenAI and Python
   ```python
from decouple import config
from openai import OpenAI
# OPENAI_API_KEY saved as env-var or defined in .env
client = OpenAI(api_key=config("OPENAI_API_KEY"))

prompt = "Describe the effects of vitamin B-12 on health."

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a nutritional expert."},
        {"role": "user", "content": prompt}
    ]
)

print(response.choices[0].message.content)
   ```

3. Learn Prompt Engineering.
   - *Basic techniques*: role definition, clear instructions, examples, request formatting
   - *Advanced* chain-of-thought prompting, few-shot examples, structure JSON output

4. Integrate AI into the UX
   - Decide *where* AI fits into your app, e.g. autocomplete, chat, smart search, recommendations, 
   - Consider effect of *response latency* or time-outs.
   - Handle errors and time-outs gracefully.
   - Add *security* and some form of *analytics* or *logging*.
   - Add *guardrails*
   
5. Learn to work with Custom Data
   - Retrieval-Augmented Generation (RAG): use relevant data from your own database or documents as input into AI at runtime
   - Use *Vector Databases* for semantic search, such as Pinecone, Weaviate, Qdrant, Milvus
   - Example: user asks a question, you find relevant docs, feed docs into AI prompt along with user question

6. Fine-Tuning and Model Hosting.  For more control,
   - fine-tune models on your domain-specific data
   - self-host an open-source model such as Hugging Face, Ollama, TensorFlow Serviing

7. Build in Security & Privacy

8. Cost Control
   - Understand the costs and what usage drives cost
   - Monitor usage and performance
   - Cache responses to reduce API costs

9. Build, Test, Iterate
   - Start with one small AI-powered feature
   - Test with real users. Adjust prompts, data retrieval, and UX.


## AI API Providers

| Provider       | API Portal                                |
|----------------|-------------------------------------------|
| OpenAI         | https://platform.openai.com/docs/overview |


## Getting an OpenAI API Key

To get an **OpenAI API key**, follow these steps:

1. Sign Up or Log In to OpenAI
   - Go to the [OpenAI website](https://openai.com/).
   - Click on "Log in" and choose "API Platform".

2. Access the API Key Section
   - Once logged in, go to the [OpenAI API Dashboard](https://platform.openai.com/).
   - Click on your **profile icon** (top-right corner) and select **"View API keys"**.

3. Create a New API Key
   - Click "Create new secret key".
   - Give it a name (optional) and click **"Create secret key"**.
   - **Copy the key immediately** - you won’t see it again after closing the pop-up.

4. Secure Your API Key
   - Store the key securely (e.g., in a password manager, KeyStore).
   - Never share it publicly (e.g., GitHub repo) and never embed it in code.

5. Use the API (Python)
   - In a virtual env, install the `openai` extension (`pip install openai`)
   - Use the key in code:
     ```python
     from decouple import config
     import openai

     openai.api_key = config("OPENAI_KEY")  # get key from env-var or .env file
     ```

Important Notes:
- OpenAI offers **free credits** for new users, but **API usage is paid** beyond that.
- Check pricing at [OpenAI Pricing](https://openai.com/pricing).
- If you lose the key, generate a new one and revoke the old one.

