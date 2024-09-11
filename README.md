# Scholaris


<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

> [!WARNING]
>
> This library is under active development and is not yet ready for
> production use. Report any issues or feature requests on the [GitHub
> repository](https://github.com/nicomarr/scholaris/issues).

## Installation

**Step 1.** [Download Ollama](https://ollama.com/download) and follow
the instructions to install Ollama for your operating system. Then, pull
and run [llama3.1](https://ollama.com/library/llama3.1) (parameters: 8B,
quantization: Q4_0, size: 4.7 GB) according to the ollama documentation.

**Step 2.** Setup a new virtual environment, such as with Conda. Quick
command line instructions on how to install Miniconda, a free minimal
installer for Conda, can be found
[here](https://docs.anaconda.com/free/miniconda/#quick-command-line-install).

``` sh
$ conda create -n scholaris-env python=3.12
```

**Step 3.** Activate the virtual environment:

``` sh
$ conda activate scholaris-env
```

**Step 4.** Install latest scholaris Python package from the GitHub
[repository](https://github.com/nicomarr/scholaris):

``` sh
$ pip install git+https://github.com/nicomarr/scholaris.git
```

or from [pypi](https://pypi.org/project/scholaris/):

``` sh
$ pip install scholaris
```

**Step 5.** Install the following dependencies:

``` sh
$ pip install ollama
$ pip install PyPDF2
$ pip install requests
$ pip install tqdm
```

## Getting started

Import the scholaris core module. If you work from a Jupyter notebook
environment within a different directory, you may need to add the parent
directory of the current working directory to the Python system path as
shown below.

``` python
from scholaris.core import *
```

Make sure the Ollama app is installed on your local computer and Llama
3.1 8B has been downloaded and is running. Then, initialize the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class with Llama 3.1 8B, the core functions and default system message.
During initialization, messages are printed to indicate whether
credentials such as API keys and email are loaded from the environment
variables (more on that below), and whether a local data directory
already exists or has been created, like so:

``` python
assistant = Assistant()
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

> [!NOTE]
>
> For the examples shown below, a local data directory had already been
> created in the parent directory prior to initialization, and a PDF
> file was downloaded and saved to the local data directory for
> demonstration purposes. You can download the same PDF file, or
> additional/other files by running the following commands in a Jupyter
> notebook cell. Alternatively, you can also download and add files
> manually to the local data directory after initialization.
>
> ``` python
> !mkdir -p ../data
> pdf_urls = [
>     "https://df6sxcketz7bb.cloudfront.net/manuscripts/144000/144499/jci.insight.144499.v2.pdf",
>     # Add more URLs here as needed
> ]
> for url in pdf_urls:
>     !curl -o ../data/$(basename {url}) {url}
> ```
>
> To see where data are stored afer initialization, simply call the
> `dir_path` attribute of the `assistant` object, like so:
>
> ``` python
> print(assistant.dir_path)
> ```

You can also explicitly set the model by passing the model name as an
argument to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class. If you do not pass a model name, the default model is ***Llama
3.1*** 8B. An alternative model that supports tool calling and can be
run on a standard laptop is Mistral’s ***NeMo*** 12 B model. To use the
latter model, change the attribute in the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class to “mistral-nemo”. For more information, read the following [blog
post](https://ollama.com/blog/tool-support).

``` python
assistant = Assistant(model="llama3.1:latest")
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

## How to use

To chat with the assistant, simply call the `chat()` method with your
prompt as input. You also can store the response in a variable, but this
is optional. By default, the assistant will stream the response and
store the conversation history.

``` python
response = assistant.chat("Tell me about the tools you have available.")
```

    I'm happy to chat with you! I have a range of tools at my disposal, designed to assist with various tasks.

    Firstly, I can summarize local documents (PDFs, markdown files, or text documents) using the `summarize_local_document` tool. This function will extract key information and provide a concise summary of the content.

    I also have access to the `describe_tools` tool, which provides an overview of my available tools and capabilities. Feel free to ask me about specific tools if you're unsure what they do!

    Additionally, I can help with tasks related to Python code using the `describe_python_code` tool. This function will provide a description of the purpose and structure of Python code in local files.

    If you need information on research articles, I have access to tools that can retrieve metadata from OpenAlex (the `query_openalex_api` tool) or Semantic Scholar (the `query_semantic_scholar_api` tool). These functions use article titles, PubMed IDs, or digital object identifiers as query parameters.

    I also have a tool for converting various types of research IDs (ID_converter_tool), which can convert between PMIDs, PMCIDs, DOIs, and manuscript IDs.

    Lastly, I can respond to generic questions using the `respond_to_generic_queries` function if no other specific tools are available.

    That's an overview of my capabilities! If you have any specific questions or requests, feel free to ask.

> [!TIP]
>
> You can also access the assistant’s responses from the `message`
> attribute, like so:
>
> ``` python
> assistant.messages[-1]["content"] # Access the last message
> ```

``` python
response == assistant.messages[-1]["content"]
```

    True

By setting `show_progress=True`, you can see the step-by-step progress
of the fuction calling. This includes the tool choice, selected
arguments, if applicable, and the output of the called function that is
being passed back to the LLM to generate the final response.

``` python
response = assistant.chat("Which PDF files do you have access to in the local data directory", show_progress=True)
```

    Selecting tools...

    [{'function': {'name': 'get_file_names', 'arguments': {'ext': 'pdf'}}}]
    Calling get_file_names() with arguments {'ext': 'pdf'}...

    Generating final response...
    Based on the available tools, I have access to 1 PDF file:

    *jci.insight.144499.v2.pdf*

    This file is located in the local data directory and has a .pdf extension. If you'd like me to summarize this document or describe its content, please let me know!

By default, streaming is enabled. If you like to disable streaming, set
`stream=False`. This will store the entire conversation history in the
`messages` attribute, which can be accessed as shown above.

``` python
response = assistant.chat("Does this document have a title?", stream_response=False)
```

    Extracting titles and first authors: 100%|██████████| 1/1 [00:07<00:00,  7.59s/it]



    Yes, this document has a title.

    The title of the PDF file *jci.insight.144499.v2.pdf* is:

    "Distinct antibody repertoires against endemic human coronaviruses in children and adults"

    This information was retrieved using the `get_titles_and_first_authors` tool.

## Conversation history

Show the conversation history by calling the assistant’s
`show_conversation_history()` method:

``` python
assistant.show_conversion_history()
```

    User: Tell me about the tools you have available.

    Assistant response: I'm happy to chat with you! I have a range of tools at my disposal, designed to assist with various tasks.

    Firstly, I can summarize local documents (PDFs, markdown files, or text documents) using the `summarize_local_document` tool. This function will extract key information and provide a concise summary of the content.

    I also have access to the `describe_tools` tool, which provides an overview of my available tools and capabilities. Feel free to ask me about specific tools if you're unsure what they do!

    Additionally, I can help with tasks related to Python code using the `describe_python_code` tool. This function will provide a description of the purpose and structure of Python code in local files.

    If you need information on research articles, I have access to tools that can retrieve metadata from OpenAlex (the `query_openalex_api` tool) or Semantic Scholar (the `query_semantic_scholar_api` tool). These functions use article titles, PubMed IDs, or digital object identifiers as query parameters.

    I also have a tool for converting various types of research IDs (ID_converter_tool), which can convert between PMIDs, PMCIDs, DOIs, and manuscript IDs.

    Lastly, I can respond to generic questions using the `respond_to_generic_queries` function if no other specific tools are available.

    That's an overview of my capabilities! If you have any specific questions or requests, feel free to ask.

    User: Which PDF files do you have access to in the local data directory

    Assistant response: Based on the available tools, I have access to 1 PDF file:

    *jci.insight.144499.v2.pdf*

    This file is located in the local data directory and has a .pdf extension. If you'd like me to summarize this document or describe its content, please let me know!

    User: Does this document have a title?

    Assistant response: Yes, this document has a title.

    The title of the PDF file *jci.insight.144499.v2.pdf* is:

    "Distinct antibody repertoires against endemic human coronaviruses in children and adults"

    This information was retrieved using the `get_titles_and_first_authors` tool.

> [!NOTE]
>
> The `show_conversation_history()` method can be called with the
> `show_function_calls=True` argument to display the function calls made
> by the assistant during the conversation, and the output of the
> function calls. This can be useful for understanding the assistant’s
> responses, and for debugging purposes.
>
> ``` python
> assistant.show_conversion_history(show_function_calls=True)
> ```

Clear the conversation history by calling the assistant’s
`clear_conversation_history()` method:

``` python
assistant.clear_conversion_history()
```

## Assistant parameters

Show the model by printing the assistant object:

``` python
assistant
```

    Assistant, powered by llama3.1

Or show the model by accessing the assistant’s model attribute:

``` python
assistant.model
```

    'llama3.1:latest'

Show the system messages by accessing the assistant’s sys_message
attribute:

``` python
print(assistant.sys_message)
```

    You are an AI assistant specialized in analyzing research articles.
            Your role is to provide concise, human-readable responses based on information from tools and conversation history.

            Key instructions:
            1. Use provided tools to gather information before answering.
            2. Interpret tool results and provide clear, concise answers in natural language.
            3. If you can't answer with available tools, state this clearly.
            4. Don't provide information if tool content is empty.
            5. Never include raw JSON, tool outputs, or formatting tags in responses.
            6. Format responses as plain text for direct human communication.
            7. Use clear formatting (e.g., numbered or bulleted lists) when appropriate.
            8. Provide article details (e.g., DOI, citation count) in a conversational manner.

            Act as a knowledgeable research assistant, offering clear and helpful information based on available tools and data.
            

## Local data access

By default, the assistant has access to a single directory, called
`data`. Within this directory, the assistant can list and read the
following file formats and extensions: pdf, txt, md, markdown, csv, and
py. If not already present, the directory is created when the assistant
is initialized. If you want to change the directory name, you can do so
by passing the desired directory name as an argument to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class when it is initialized. For example, to create a directory called
`proprietary_data`, you would initialize the assistant as follows:

``` python
assistant = Assistant(dir_path="../proprietary_data")
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/proprietary_data already exists for storing data files. No of files: 0

## Tools

By default, the assistant can call a set of core tools or functions
which are passed to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
as a dictionary when it is initialized. With these tools or functions,
the assistant will be able to get a list of file names in a specific
data directory, can extract content from these files, and summarize
them. In addition, the assistant can make API calls to external data
sources, such as [OpenAlex](https://openalex.org) or [Semantic
Scholar](https://www.semanticscholar.org), to retrieve information about
a large number of scholarly articles. The tools available to the
assistant can be viewed by accessing the assistant’s `list_tools()`
method as follows:

``` python
assistant.list_tools()
```

    get_file_names
    extract_text_from_pdf
    get_titles_and_first_authors
    summarize_local_document
    describe_python_code
    id_converter_tool
    query_openalex_api
    query_semantic_scholar_api
    respond_to_generic_queries
    describe_tools

> [!TIP]
>
> You can learn more details about the core tools by visiting the Source
> Code page, which lists each function and provides a brief description
> of its purpose, functionality, required arguments, and usage (the
> docstring). This information helps you understand the available tools
> and how the LLM uses them. Alternatively, you can execute the
> `assistant.pprint_tools()` or `assistant.get_tools_schema()` methods.

## Authentication for tool calling and API access (optional)

Some tools take optional authentication parameters, such as an API key
or email. For example, the `query_semantic_scholar` tool takes an
optional API key to access the Semantic Scholar API, which will increase
the API rate limit. Request a Semantic Scholar API Key
[here](https://www.semanticscholar.org/product/api/tutorial). Similarly,
the `query_openaplex_api` tool takes an optional email parameter to
access the OpenAlex API, which is recommended as a best practice and
kindly requested by the [API
provider](https://docs.openalex.org/how-to-use-the-api/rate-limits-and-authentication#the-polite-pool).

The best way to pass these parameters is to set them as environment
variables, with the following key names: `SEMANTIC_SCHOLAR_API_KEY` and
`EMAIL`. The Assistant class will automatically read these environment
variables when initialized and pass them to the tools that require them.
Alternatively, you can pass the Semantic Scholar API key and your email
by simply adding the authentication argument when initializing the
Assistant class, as shown below:

``` python
authentication = {
    "SEMANTIC_SCHOLAR_API_KEY": "your_api_key",
    "EMAIL": "your_email@example.com"
}
assistant = Assistant(authentication=authentication)
```

    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

If you want to change the core functions, you can do so by passing the
desired core functions as an argument to the Assistant class when it is
initialized. For example, to limit the assitant’s ability to respond to
generic questions and to access external data by making requests to the
OpenAlex and Semantic Scholar API’s, you would initialize the assistant
as follows:

``` python
assistant = Assistant(tools = {
    "query_openalex_api": query_openalex_api,
    "query_semantic_scholar_api": query_semantic_scholar_api,
    "respond_to_generic_queries": respond_to_generic_queries,
    })
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

> [!NOTE]
>
> The research assistent is set up so that it has to use a tool to
> generate a final response to a user’s prompt. This is to ensure that
> the assistant is primarily providing information which is relevant for
> health and life sciences. Otherwise it will abort the conversation,
> like so:

``` python
assistant = Assistant(tools = {})
assistant.chat("What is the capital of France?")
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

    No tools provided! Please add tools to the assistant.
    No tool calls found in the response. Adding an empty tool_calls list to the conversation history. Aborting...

:::{.callout-tip} When working in a Jupyter notebook environment, you
can quickly display details of the class methods by using special
syntax. Type the name of the object, followed by a `?`, or type `??` to
get more detailed information (i.e., the source code), like so:

``` python
assistant.chat?
assistant??
```

## Developer Guide

### Defining new tools

You can define new functions to be used by the assistant as tools. To
simplify this process, a decorator called `@json_schema_decorator` is
provided so it is not necessary to define the schema for the function.
The schema is automatically generated based on the function’s annotation
and docstring.

> [!TIP]
>
> Here are key points to consider when defining a new tool:
>
> - Use type hints in the function signature to define the input and
>   output types.
> - Use docstrings, as shown below, to describe the function’s purpose
>   and the expected input and output.
> - Use the `@json_schema_decorator` decorator to automatically generate
>   the schema for the function.
> - Ensure the output is a string (such as a JSON-formatted string) that
>   can be passed back to the LLM to generate the final response.

> [!IMPORTANT]
>
> It’s crucial to understand that this metadata (function name, type
> hints, and docstring) is all the information the LLM has access to
> when deciding which function to call and how to use it. The LLM does
> not have access to or information about the actual source code or
> implementation of the functions (unless explicitly provided).
> Therefore, the metadata must be comprehensive and accurate to ensure
> proper function selection and usage by the LLM.

The following example shows how to define a new tool called
`multiply_two_integers` that takes two integers as input and returns a
string as output:

``` python
@json_schema_decorator
def multiply_two_integers(a: int, b: int) -> int:
    """
    A function to multiply two numbers.

    Args:
        a (int): First integer.
        b (int): Second integer.

    Returns:
        str: The product of the two numbers, as a string.
    """

    if not isinstance(a, int) or not isinstance(b, int):
        return "Error: Inputs must be integers."
    
    return str(a * b)
```

To ensure the JSON schema is generated correctly, you can call the
`json_schema` attribute of the function:

``` python
multiply_two_integers.json_schema
```

    {'type': 'function',
     'function': {'name': 'multiply_two_integers',
      'description': 'A function to multiply two numbers.',
      'parameters': {'type': 'object',
       'properties': {'a': {'type': 'integer', 'description': 'First integer.'},
        'b': {'type': 'integer', 'description': 'Second integer.'}},
       'required': ['a', 'b']}}}

Alternatively, you can use the
[`generate_json_schema`](https://nicomarr.github.io/scholaris/core.html#generate_json_schema)
function:

``` python
generate_json_schema(multiply_two_integers)
```

    {'type': 'function',
     'function': {'name': 'multiply_two_integers',
      'description': 'A function to multiply two numbers.',
      'parameters': {'type': 'object',
       'properties': {'a': {'type': 'integer', 'description': 'First integer.'},
        'b': {'type': 'integer', 'description': 'Second integer.'}},
       'required': ['a', 'b']}}}

### Adding tools

You can add new tools by passing a dictionary of new tools to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class when it is initialized. Use the `add_tools` argument to add new
tools to the assistant. This will merge the new tools with the existing
tools. For example, to add the new tool called `multiply_two_integers`
to the assistant, you would initialize the assistant as follows:

``` python
assistant = Assistant(add_tools = {"multiply_two_integers": multiply_two_integers})
```

    Loaded Semantic Scholar API key from the environment variables.
    Loaded email address from the environment variables.
    A local directory /Users/user2/GitHub/scholaris/data already exists for storing data files. No of files: 1

You can confirm that the new tool has been added to the list of existing
tools by using the `list_tools()` method:

``` python
assistant.list_tools()
```

    get_file_names
    extract_text_from_pdf
    get_titles_and_first_authors
    summarize_local_document
    describe_python_code
    id_converter_tool
    query_openalex_api
    query_semantic_scholar_api
    respond_to_generic_queries
    describe_tools
    multiply_two_integers

``` python
assistant.chat("What is the product of 4173 and 351?", show_progress=True)
```

    Selecting tools...

    [{'function': {'name': 'multiply_two_integers', 'arguments': {'a': '4173', 'b': '351'}}}]
    Calling multiply_two_integers() with arguments {'a': '4173', 'b': '351'}...

    Generating final response...
    To calculate the product, I'll treat the inputs as integers.

    The product of 4173 and 351 is 1,464,303.

    "To calculate the product, I'll treat the inputs as integers.\n\nThe product of 4173 and 351 is 1,464,303."

### Adding methods

You can add new methods to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class by using the `add_to_class()` decorator function. For example, the
[`clear_conversion_history`](https://nicomarr.github.io/scholaris/core.html#clear_conversion_history)
method was added to the
[`Assistant`](https://nicomarr.github.io/scholaris/core.html#assistant)
class as follows:

``` python
@add_to_class(Assistant)
def clear_conversion_history(self):
    """Clear the conversation history."""
    self.messages = [{'role': "system", 'content': self.sys_message},]
```

### Contributing to this library

This library has been developed using the
[nbdev](https://nbdev.fast.ai/) framework. To contribute to this
library, install `nbdev` and follow the [nbdev
documentation](https://nbdev.fast.ai/tutorials/tutorial.html) to set up
your development environment.

## Acknowledgements

Many thanks to the developers of [Ollama](https://ollama.com/) and the
[Ollama Python library](https://pypi.org/project/ollama/) for providing
the core functionality that Scholaris is built upon. Special thanks to
the developers of [nbdev](https://nbdev.fast.ai/) for making it easy to
develop and document this library, and for many insightful tutorials and
inspirations!
