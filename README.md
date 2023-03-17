# GPT-Azure-Search-Engine 
### (Azure Cognitive Search + Azure OpenAI Accelerator)

![Architecture](GPT-Smart-Search-Architecture.jpg "Architecture")

## ** Demo **

https://webapp-bbuqqkw6y2eee.azurewebsites.net/

## 🔧**Features**

   - Shows how you can use [Azure OpenAI](https://azure.microsoft.com/en-us/products/cognitive-services/openai-service/) + [Azure Cognitive Search](https://azure.microsoft.com/en-us/products/search) to have a Smart and Multilingual Search engine that not only provides links of the search results, but also answers the question.
   - ***Solve 80% of the use cases where companies want to use OpenAI to provide answers from their knowledge base to customers or employees, without the need of retraining and hosting the models.***
   - All Azure services and configuration are deployed via python code.
   - Uses [Azure Cognitive Services](https://azure.microsoft.com/en-us/products/cognitive-services/) to enrich documents: Detect Language, OCR images, Key-phrases extraction, entity recognition (persons, emails, addresses, organizations, urls).
   - Uses [LangChain](https://langchain.readthedocs.io/en/latest/) as a wrapper for interacting with Azure OpenAI , vector stores and constructing prompts.
   - Uses [Streamlit](https://streamlit.io/) to build the web application in python.
   - (Coming soon) recommends new searches based on users' history.

## **Steps to Run the Accelerator**

Note: (Pre-requisite) You need to have an Azure OpenAI service already created

1. Clone this repo to your Github account.
2. In Azure OpenAI studio, deploy these two models: Make sure that the deployment name is the same as the model name.
   - "text-davinci-003"
   - "text-embedding-ada-002"
3. Create a Resource Group where all the assets of this accelerator are going to be.
4. Create an Azure Cognitive Search Service and Cognitive Services Account by clicking below: \<br\>

[![Deploy To Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpablomarin%2FGPT-Azure-Search-Engine%2Fmain%2Fazuredeploy.json) 

_Note: If you have never created a cognitive multi-service account before, please create one manually in the azure portal to read and accept the Responsible AI terms. Once this is deployed, delete this and then use the above deployment button._

5. Enable Semantic Search on your Azure Cognitive Search Service:
   - On the left-nav pane, select Semantic Search (Preview).
   - Select either the Free plan or the Standard plan. You can switch between the free plan and the standard plan at any time.
6. Install the dependencies on your machine:
```
pip install -r ./requirements.txt
```
7. Edit app/credentials.py with your azure services information
8. Run 01-Load-Data-ACogSearch.ipynb:
   - Loads data into your Search Engine and create the index with AI skills
9. Run 02-Quering-AOpenAI.ipynb and:
   - Run queries in Azure Cognitive Search and see how they compare with enhancing the experience with Azure OpenAI
10. Go to the app/ folder and click the Deploy to Azure function to deploy the Web Application in Azure Web App Service. It takes a few minutes.
   - The deployment automatically comes with CI/CD, so any change that you commit/push to the code will automatically trigger a deployment in the Application.

## **FAQs**

1. Why the vector similarity is done in memory using FAISS versus having a separate vector database like RedisSearch or Pinecone?
A: True, doing the embeddings of the documents pages everytime that there is a query is not efficient. The ideal scenario is to vectorize the docs pages once (first time they are needed) and then retrieve them from a database the next time they are needed. For this a special vector database is necessary. The ideal scenario though, is Azure Search to retreive the vectors as part of the search results, along with the document pages. Azure Search will soon allow this in a few months, let's wait for it. As of right now the embedding process doesn't take that much time or money, so it is worth the wait versus using another database just for vectors.

2. Why use the REFINE type in LangChaing versus STUFF type? 
A: Because using STUFF type with all the content of the pages as context, uses too many tokens. So the best way to deal with large documents is to refine the answers by going trough all of the search results and do many calls to the LLM looking for a refined answer. For more information of the difference between STUFF and REFINE, see [HERE](https://langchain.readthedocs.io/en/latest/modules/indexes/combine_docs.html)

3. Why use Azure Cognitive Search engine to provide the context for the LLM and not fine tune the LLM instead?
A: Quoting the [OpenAI documentation](https://platform.openai.com/docs/guides/fine-tuning): "GPT-3 has been pre-trained on a vast amount of text from the open internet. When given a prompt with just a few examples, it can often intuit what task you are trying to perform and generate a plausible completion. This is often called "few-shot learning.
Fine-tuning improves on few-shot learning by training on many more examples than can fit in the prompt, letting you achieve better results on a wide number of tasks. Once a model has been fine-tuned, you won't need to provide examples in the prompt anymore. This saves costs and enables lower-latency requests"

So training/fine tunning the model requires that we provide hundreds/thousands of Prompt and Completion tuples, or in other words we need to provide samples of query-responses. For a company knowledge base of Terabytes of information this is not feasible. To come up with all the possible tuples that users my request, is simply not possible. So the search engine is absolutely necessary for a company data search engine using OpenAI.

## **Known Issues**
1. Error when clicking BEST ANSWER on the App: "This model's maximum context length is 2047 tokens, however you requested xxxx tokens (xxxxx in your prompt; 0 for the completion). Please reduce your prompt; or completion length"
This error happens if your embedding model text-embedding-ada-002 has a limit of 2047. Older versions of this model in Azure OpenAI has this reduced limit. However the newer versions has the 8147 limit. Make sure you request the newer version, or if not possible, reduce the size of the TextSplit in Azure Search indexing from 5000 (default) to 3500.

2. When deploying the App on Azure portal using the ARM template: "Cannot find SourceControlToken with name Github"
This error happens randomly and means that Azure Web Services cannot connect to your Github repo for CI/CD. 
If you encounter this error, you can  delete the deployment and try it again, or, if it keeps happening, try to connect the app to Github by going to the Azure Portal, going to the newly deploy Web App, go to Deployment Center->Settings and connect the app to your Github repo.


