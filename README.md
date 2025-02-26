[![DOI](https://zenodo.org/badge/533971880.svg)](https://zenodo.org/badge/latestdoi/533971880)

# `rgpt3` 

**Making requests from R to the GPT-3 API and to ChatGPT**


_Note: this is a "community-maintained” package (i.e., not the official one). For the official OpenAI libraries (python and node.js endpoints), go to [https://beta.openai.com/docs/libraries/python-bindings](https://beta.openai.com/docs/libraries/python-bindings)_

## Getting started

You can follow these steps to get started with making requests and retrieving embeddings from the Open AI GPT-3 language model.

_If you already have an Open AI API key, you can skip step 1._

1. Obtain your API key

Go to [https://openai.com/api/](https://openai.com/api/), register for a free account and obtain your API key located at [https://beta.openai.com/account/api-keys](https://beta.openai.com/account/api-keys).

Note that Open AI may rotate your key from time to time (but you should receive an email on your registered account if they do so).

2. Set up the `access_key.txt` file

Your access workflow for this package retrieves your API key from a local file. That file is easiest called `access_key.txt` (but any other file name would do). Important is that you use a `.txt` file with just the API key in there (i.e. no string quotation marks).

The path to that file (e.g. `/Users/me/directory1/access_key.txt`) is needed for the `gpt3_authenticate()` function (see below).

When using a version control workflow, make sure `access_key.txt` is in the `.gitignore` file (i.e., so your access code is not visible on a GitHub repo). If you use the package without any version control, you do not need to set a `.gitignore` file.

3. Install the `rgpt3` package

The easiest way to use the package (before its CRAN release) is:

```{r}
devtools::install_github("ben-aaron188/rgpt3")
library(rgpt3)
```

4. Run the test workflow

Once the package is installed, you will typically run this work flow:

**Authentication:**

Get the path to your access key file and run the authentication with: `gpt3_authenticate("PATHTO/access_key.txt")`

**Make the test request:**

You can run the test function below, which sends a simple request (here: the instruction to "Write a story about R Studio:") to the API and returns the output in the format used in this package (i.e., `list[[1]]` --> prompt and output, `list[[2]]` = meta information).

```{r}
gpt3_test_completion()
```

## Known issues and fixes

### API call error

Error: `Error in core_output$gpt3[i] ... replacement has length zero`

This error is telling you that the call made to the API was not successful. This should already fail when you run the `gpt3_test_completion()` function. There are several things that can cause this, so best check the below if you get this error (in that order):

1. Did you run the authentication properly to your API key with `gpt3_authenticate("/Users/.../access_key.txt")` (the path can take any form as long as you point it to a file (here called: `access_key.txt`) where your API key is located
1. Does the file with your API key (e.g., `access_key.txt`) contain a newline after the API? It should contain a newline, so add one if this is missing.
1. Is your free API limit from OpenAI still within the valid period (this is usually one month)?
1. Are you still within the actual API limit (whether you use a free account or a paid one)?

If you can answer "yes" to all of the above and the error persists, then please [open an issue](https://github.com/ben-aaron188/rgpt3/issues) so this can be looked into.





## Core functions

`rgpt3` currently is structured into the following functions:

- Making _standard_ requests (i.e. prompting the GPT models)
    - single requests: `gpt3_single_completion()`
    - make multiple prompt-based requests from a source data.frame or data.table: `gpt3_completions()`
- Interacting with ChatGPT
    - single chat requests: `chatgpt_single()`
    - make multiple prompt-based chat requests from a source data.frame or data.table: `chatgpt()`
- Obtain embeddings
    - obtain embeddings for a single text input: `gpt3_single_embedding`
    - obtain embeddings for multiple texts from a source data.frame or data.table: `gpt3_embeddings()`

The basic principle is that you can (and should) best use the more extensible `gpt3_completions()`, `chatgpt()` and `gpt3_embeddings()` functions as these allow you to make use of R's vectorisation. These do work even if you have only one prompt or text as input (see below). The difference between the extensible functions and their "single" counterparts is the input format.

This R package gives you full control over the parameters that the API contains. You can find these in detail in the package documentation and help files (e.g., `?gpt3_completions`) on the Open AI website for [completion requests](https://beta.openai.com/docs/api-reference/completions/create) and [embeddings](https://beta.openai.com/docs/api-reference/embeddings/create).

Note: this package enables you to use the core functionality of GPT-3 (making completion requests) and ChatGPT, and provides a function to obtain embeddings. There are additional functionalities in the core API such as fine-tuning models (i.e., providing labelled data to update/retrain the existing model) and asking GPT-3 to make edits to text input. These are not (yet) part of this package since the focus of making GPT-3 accessible from R is on the completion requests.


## Examples

The examples below illustrate all functions of the package.

Note that due to the [sampling temperature parameter](https://beta.openai.com/docs/api-reference/completions/create#completions/create-temperature) of the requests - unless set to `0.0` - the results may vary (as the model is not deterministic then).

### Making requests (standard GPT models, i.e. before ChatGPT)

The basic form of the GPT-3 API connector is via requests. These requests can be of various kinds including questions ("What is the meaning of life?"), text summarisation tasks, text generation tasks and many more. A whole list of examples is on the [Open AI examples page](https://beta.openai.com/examples).

Think of requests as instructions you give the to model. You may also hear the instructions being referred to as _prompts_ and the task that GPT-3 then fulfills as _completions_ (have a look at the API guide on [prompt design](https://beta.openai.com/docs/guides/completion/prompt-design)).

**Example 1: making a single completion request**

This request "tells" GPT-3 to write a cynical text about human nature (five times) with a sampling temperature of 0.9 and a maximium length of 100 tokens.

```{r}
example_1 = gpt3_single_completion(prompt_input = 'Write a cynical text about human nature:'
                    , temperature = 0.9
                    , max_tokens = 100
                    , n = 5)
```

The returned list contains the actual instruction + output in `example_1[[1]]` and meta information about your request in `example_1[[2]]`.


**Example 2: multiple prompts**

We can extend the example and make multiple requests by using a data.frame / data.table as input for the `gpt3_completions()` function:

```{r}
my_prompts = data.frame('prompts' = 
                          c('Complete this sentence: universities are'
                            , 'Write a poem about music:'
                            , 'Which colour is better and why? Red or blue?')
                        ,'prompt_id' = c(LETTERS[1:3]))

example_2 = gpt3_completions(prompt_var = my_prompts$prompts
              , id_var = my_prompts$prompt_id
              , param_model = 'text-curie-001'
              , param_max_tokens = 100
              , param_n = 5
              , param_temperature = 0.4)
```

Note that this completion request produced 5 (`param_n = 5`) completions for each of the three prompts, so a total of 15 completions.


### Embeddings

Here we load the data that comes with the package:

```{r}
data("travel_blog_data")
```

The dataset is now available in your environment as `travel_blog_data`. The column (`travel_blog_data$gpt3`) contains ten texts written by GPT-3 (instructed to: "Write a travel blog about a dog's journey through the UK:").


**Example 1: get embeddings for a single text**

We can ask the model to retrieve embeddings for [text similarity](https://beta.openai.com/docs/guides/embeddings/similarity-embeddings) for a single text as follows:

```{r}
# we just take the first text here as a single text example
my_text = travel_blog_data$gpt3[1]
my_embeddings = gpt3_single_embedding(input = my_text)
length(my_embeddings)
# 1024 (since the default model uses the 1024-dimensional Ada embeddings model)

```


**Example 2: get embeddings from text data in a data.frame / data.table**

Now we can do the same example with the full `travel_blog_data` dataset:


```{r}
multiple_embeddings = gpt3_embeddings(input_var = travel_blog_data$gpt3
                                      , id_var = travel_blog_data$n)
dim(multiple_embeddings)

# [1]    10 1025 # because we have ten rows of 1025 columns each (by default 1024 embeddings elements and 1 id variable)
```

## ChatGPT

In principle, ChatGPT can do all the things that the _standard_ GPT models (e.g., DaVinci-003) can do, but just a little better. An excellent brief summary is provided by OpenAI here: [https://openai.com/blog/chatgpt](https://openai.com/blog/chatgpt). In the examples below, we will reproduce the ones from the `gpt3_single_completion()` and `gpt3_completions()` as listed above. 

The biggest change in how we interact with ChatGPT's API compared to the previous models is that we send the requests with a **role** and **content** of the prompts. The role must be one of 'user', 'system' or 'assistant' and you essentially tell ChatGPT in wich role the content you send is to be interpreted. The content is analogous to the standard prompts. The reason why the role is necessary is that it allows you provide a full back-and-forth conversational flow (e.g., [https://platform.openai.com/docs/guides/chat/introduction](https://platform.openai.com/docs/guides/chat/introduction)).

**Example 1: making a single chat completion request**

This request "tells" ChatGPT to write a cynical text about human nature (five times) with a sampling temperature of 1.5 and a maximium length of 100 tokens.

```{r}
chatgpt_example_1 = gpt3_single_completion(prompt_input = 'Write a cynical text about human nature:'
                    , temperature = 0.9
                    , max_tokens = 100
                    , n = 5)
```

The returned list contains the actual instruction + output in `chatgpt_example_1[[1]]` and meta information about your request in `chatgpt_example_1[[2]]`.

A verbatim excerpt of the produced output (from the `chatgpt_example_1[[1]]$chatgpt_content` column) here is: 

> Settle in, young one. Let me impart to you some hard-earned wisdom about human nature. It is a wild creature, not easily tamed or discernible. Some claim they understand it fully, but they are as deluded as a chimpanzee wearing a top hat. I've seen people  at their worst: manipulating, deceiving, and diminishing others simply to assert their misguided superiority. [...]


**Example 2: multiple prompts**

We can extend the example and make multiple requests by using a data.frame / data.table as input for the `chatgpt()` function:

```{r}
my_chatgpt_prompts = data.frame('prompts_roles' = rep('user', 3)
                          , 'prompts_contents' =
                            c('You are a bureacrat. Complete this sentence: universities are'
                              , 'You are an award-winning poet. Write a poem about music:'
                              , 'Which colour is better and why? Red or blue?')
                        ,'prompt_id' = c(LETTERS[1:3]))

chatgpt_example_2 = chatgpt(prompt_role_var = my_chatgpt_prompts$prompts_roles
                            , prompt_content_var = my_chatgpt_prompts$prompts_contents
                             , id_var = my_chatgpt_prompts$prompt_id
                             , param_max_tokens = 100
                             , param_n = 5
                             , param_temperature = 0.4)
```

Note that this completion request produced 5 (`param_n = 5`) completions for each of the three prompts, so a total of 15 completions.



## Cautionary note

**Read this:** using GPT-3 is not free, so any interaction with the GPT-3 model(s) is counted towards your token quota. You can find details about Open AI's pricing model at [https://openai.com/api/pricing/](https://openai.com/api/pricing/).

You receive a reasonable credit for free and do not need to provide any payment information for the first interactions with the model(s). Once your token quota nears its end, Open AI will let you know. Your usage is traceable in your Open AI account dashboard.


## Support and contributions

If you have questions or problems, please raise an issue on GitHub.

You are free to make contributions to the package via pull requests. If you do so, you agree that your contributions will be licensed under the [GNU General Public License v3.0](https://github.com/ben-aaron188/rgpt3/blob/main/LICENSE.md).

## Changelog/updates

- [new release] 5 Mar 2023: the package now supports ChatGPT 
- [update] 30 Jan 2023: added error shooting for API call errors
- [update] 23 Dec 2022: the embeddings functions now default to the second generation embeddings "text-embedding-ada-002".
- [minor fix] 3 Dec 2022: included handling for encoding issues so that `rbindlist` uses `fill=T` (in `gpt3_completions(...)`)
- [update] 29 Nov 2022: the just released [davinci-003 model](https://beta.openai.com/docs/models/gpt-3) for text completions is now the default model for the text completion functions.

## Citation

```
@software{Kleinberg_rgpt3_Making_requests_2022,
    author = {Kleinberg, Bennett},
    doi = {10.5281/zenodo.7327667},
    month = {3},
    title = {{rgpt3: Making requests from R to the GPT-3 API and ChatGPT}},
    url = {https://github.com/ben-aaron188/rgpt3},
    version = {0.4},
    year = {2023}
}
```

See also "Cite this repository" on the right-hand side.
