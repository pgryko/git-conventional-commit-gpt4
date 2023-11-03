Git commit logs can be tedious to write...

![<img src="https://xkcd.com/1296/">](https://imgs.xkcd.com/comics/git_commit.png)

but are useful for long term maintenance and code audit.

Turns out that GPT4 is fantastic for generating git logs from git diffs, making the process a breeze.

The following shell script gets staged code and generates a diff and calls the GTP4 api to write a git log:

```shell
#!/bin/bash

diff_content="Write a commit message in the style of conventional commits specification, using bullet points, for the following: \n $(git --no-pager diff --cached)"

echo $diff_content

payload=$(jq -nc --arg content "$diff_content" '{
    "model": "gpt-4",
    "messages": [
        {"role": "system", "content": "You are ChatGPT, a large language model trained by OpenAI. Carefully heed the users instructions Respond using Markdown."},
        {
            "role": "user",
            "content": $content
        }
    ],
   "temperature": 1
}')

# Pass the payload to the OpenAI API chat completions endpoint
response=$(curl -s -X POST https://api.openai.com/v1/chat/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer $OPENAI_API_KEY" \
    --data "$payload")

echo $response | jq -r '.choices[0].message.content'

```

Save the file somewhere locally, e.g ~/git_diff_to_gpt4.sh, and set it to executable `chmod +x ~/git_diff_to_gpt4.sh`.
Then add to either your .zshrc or .bashrc, an alias and the openai api key as an environmental variable, i.e.

```shell
export OPENAI_API_KEY="YOUR_OPENAI_API_KEY"
alias gdiffgpt4='~/git_diff_to_gpt4.sh'
```

Then, either restart the shell or reload it (e.g. 'source ~/.zshrc'), stage your code and run the shell alias i.e.

```shell
git add .
gdiffgpt4
```

Note, this sends your git diff to OpenAI, and should not used for proprietary codebases without permission. It should be possible to modify this code to us a locally hosted LLM such as LLAMA2 or CodeLLAMA. 

![Terminal recording](/termtosvg_20y3d90v.svg)
