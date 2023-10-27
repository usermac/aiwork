# aiwork
FileMaker database to interact with OpenAI's API and get an evaluated calculation result back in real time. 

2023-10-23 - AIwork_v2.fmp12 is a Filemaker database to make OpenAI API calls. It offers you a place to put in your prompt and another field to put in *how* you want AI to respond (see last paragraph.)

You'll need a new api key. If you're selected on a record once you get your api call to work and then create a new record, it will copy over the setting as before. Welcome to change this to your liking. Sample request have been kept in the file so you can see what it does.

The most important finding is the *"Your response will be formal and terse without comments or commentary and only pure code presented in your response."* to put in the field named ai_custom_instructions_reponse. Use this when asking for a FileMaker calculation. It will respond with code that will get interpreted in field "evaluate." And yes, I know the grammar is bad but it worked so I'm keeping it.  - Brian

"Why back in my day, we used to write out our calculations" - me

https://github.com/usermac/aiwork

P.S. - No support is offered.

![AIwork](https://github.com/usermac/aiwork/assets/4897287/33b62268-8304-4839-bff2-a16b22129da3)

