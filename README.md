# Azure-AI-Agent-CodeInterpreter-Blob-File
This is to show how to use the last_msg annotation part to identify the file_id and full name of the file, therefore we can persist the file with the right extension to blob for further consumption.

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
