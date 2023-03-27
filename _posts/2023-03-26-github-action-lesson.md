---
layout: post
title: "Tips/Lessons from Github Action Practice (ENG)"
categories: cicd
author: "Yongchan Hong"
---

# Tips/Lessons from Github Action Practice (ENG)
<img src="https://cdn.invicti.com/statics/img/drive/h2jfrvzrbyh1yff2n3wfu2hkqqps6x_uvqo.png"  width="50%" height="30%">

Recently I have been working on Github Action to build Docker image and deliver the result to Slack channel, including the mention of particular user.
Here are some lessons/tips that I have learned while working on it.

- Github Action has component called `continue-on-error`. You can add this on job or particular step. By setting this component as true, you can continue on running workflow even though it has error. You can add like following:
```
jobs:
    example:
        name: example
        runs-on: ubuntu-latest
        steps:
            - name: "test1"
              continue-on-error: true
            - name: "test2" # test2 will run even if test1 fails
``` 

- You can set when particular step to run conditionally using if with an expression. For status check functions, you can use `always`, `success`, `failure` or `cancelled`. Default will be success.
```
steps:
    - name: The job is completed
      if: ${{ always() }}

    - name: Run when event is push
      if: ${{ github.event_name == 'push' }}
```

- You can extract webhook payload object. Check this [documentation](https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads) for appropriate parameters.

- In order to set environmental variable with a bash expression, you can do following:
```
- name: Set Env Example
  run: echo "EXAMPLE_TEST=hi" >> $GITHUB_ENV # You should export to Github env
- name: Read Env Example
  run: echo $EXAMPLE_TESTz
```

- You an also set up environmental variable on full workflow:
```
name: Example
on:
  push
env:
  NAME: Chan
jobs:
 ---
```


- Github Action and Slack doesn't link well. This is because github email/id is separated from Slack id. In order to connect them, you should either build your own Slack App, or you should maintain your own mapping json/yaml. I have used library called `zoexx/gihtub-action-json-file-properties` in order to extract Slack id from appropriate Github id/email. Here is an example:
```
steps:
    - name: Slack User Extraction
      if: ${{ github.event_name == 'push' }} # only run when it is pushed
      uses: zoexx/gihtub-action-json-file-properties@release
      with:
        file_path: ".github/config/slack_mapping.json"
        prop_path: "${{ github.event.head_commit.author.name }} # extract head commiter name
```
And here is `slack_mapping.json`
```
{
    "Chan": "U01XXXXXX"
}
```

- There are multiple Github Action libraries that deliver slack messages for Github Action workflows, jobs and steps. But I personally recommend [ac10ns/slack](https://github.com/act10ns/slack) since it is easily customizable. You only need SLACK_WEBHOOK_URL as an environmental variable.


### Reference
https://www.invicti.com/support/integrating-invicti-enterprise-github-actions/  
https://docs.github.com/en/webhooks-and-events/webhooks/webhook-events-and-payloads  
https://github.com/act10ns/slack  