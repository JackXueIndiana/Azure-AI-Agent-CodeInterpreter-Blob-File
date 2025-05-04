# Azure-AI-Agent-CodeInterpreter-Blob-File
AZure AI Agent tool CodeInterpreter can generate code, run code and output the result per user prompt. However, the accessibility and persistance of the results are challenge. Here we show how to leverage the last_msg annotation part to identify the file_id and full name of the file, therefore we can persist the file with the right extension to Azure blob (or any file service) for further consumption.

## Key points
- Prompt engineering
We add a sentence at the end of user prompt:
~~~
user_message = request.message + " Save the result to a file."
~~~
- Parsing the annotation
~~~
# Access the attributes of the annotation directly
                    try:
                        annotation = last_msg.text.annotations[0]
                        print(annotation)
                    except Exception as e:
                        print(f"Error: {e}")
                        return f"annotation error: {e}"
# If you need to convert the annotation to a dictionary
                    annotation_dict = {
                        "type": annotation.type,
                        "text": annotation.text,
                        "file_path": annotation.file_path.file_id if annotation.file_path else None,
                    }
                    print(json.dumps(annotation_dict, indent=2))

                    root, extension = os.path.splitext(annotation_dict["text"])
                    file_name = str(uuid.uuid4()) + extension  # Convert UUID to string
                    file_id = annotation_dict["file_path"]
                    print(f"File name: {file_name}, File ID: {file_id}")
~~~
- Save the file locally
~~~
project_client.agents.save_file(file_id=file_id, file_name=file_name)
                    print(f"Saved the file to: {file_name}")
~~~
- Push the file to blob
~~~
# save the newly created file
                    from azure.storage.blob import BlobServiceClient
                    blob_connection_string = os.getenv("AZURE_BLOB_CONNECTION_STRING")
                    blob_container_name = os.getenv("AZURE_BLOB_CONTAINER_NAME")
                    if not blob_connection_string or not blob_container_name:
                        raise ValueError("Missing required environment variables for Azure Blob Storage.")

                    blob_service_client = BlobServiceClient.from_connection_string(blob_connection_string)
                    blob_client = blob_service_client.get_blob_client(container=blob_container_name, blob=file_name)
                    with open(file_name, "rb") as data:
                        blob_client.upload_blob(data, overwrite=True)

                    # Get the full URL of the uploaded file
                    file_url = blob_client.url
~~~
## ipynb
This notebook reads in the csv file and generates a excel file with name <uuid>.xlsx in a blob container.

## py
This python class is a part of a FastAPI app, it takes an input, such as
~~~
{
  "message": "Simulate 10,000 rolls of two six-sided dice and estimate the probability distribution of their sum. Show the result in a bar chart and save it in a pdf file.",
  "thread_id": "string"
}
~~~
The output is 
~~~
return(f"{last_msg.text.value} \nA copy in [cloud]({file_url})")
~~~
A PDF file with the generated bar chart will be saved in a blob container.

For input
~~~
{
  "message": "Simulate 10,000 rolls of two six-sided dice and estimate the probability distribution of their sum. Show the result in a spreadsheet.",
  "thread_id": "string"
}
~~~
A csv file will be push to the blob container.
