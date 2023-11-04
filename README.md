# aiwork
_2023-11-04 update - And now a recovery from the low quality results of the past days. Looks like results are getting better. - Brian_

_2023-11-01 update - It appears OpenAI's model is updated from what I've read from tech news and I would agree as my results have been worse since Halloween through today. It was good while it lasted. - Brian_

FileMaker database to interact with OpenAI's API, get a pure calculation result returned, then evaluate that result in real time. 

2023-10-23 - AIwork_v2.fmp12 is a Filemaker database to make OpenAI API calls. It offers you a place to put in your prompt and another field to put in *how* you want AI to respond (see last paragraph.)

You'll need a new api key. If you're in a field on a record once you get your api call to work and then create a new record, it will copy over the setting as before. Welcome to change this to your liking. Sample request have been kept in the file so you can see what it does. Granted, many are repetitive but needed to test. 

The most important finding is the *"Your response will be formal and terse without comments or commentary and only pure code presented in your response."* to put in the field named ai_custom_instructions_reponse. Use this when asking for a FileMaker calculation. It will respond with code that will get interpreted in field "evaluate." And yes, I know the grammar is bad but it worked so I'm keeping it.  - Brian

https://github.com/usermac/aiwork

PS - No support is offered.

## Sample of actual use
The real time example was an actual live suggestion during my developer talk this past Tuesday (10/24/2023) recreated here. You can see the original here: https://youtu.be/0d9QaDj5zXI?si=w7MMFcRUABEzfHSw

![AIwork](https://github.com/usermac/aiwork/assets/4897287/33b62268-8304-4839-bff2-a16b22129da3)

## Details
### The "Ask AI" button script
```
# AI cURL OpenAI in file AIwork_v2

# /*
 * @SIGNATURE:
 * AI cURL OpenAI
 *
 * @PARAMETERS:
 *
 * @HISTORY:
 * Modified: YYYY-Mon-DD by FName LName
 * Created: 2023-10-21 by Brian Ginn brian@brianginn.com
 *
 * @PURPOSE:
 * Talk to Open AI via API to get results within FileMaker using the current record.
 *
 * @RESULT:
 * Raw response from OpenAI.
 *
 * @ERRORS:
 * List any errors that may be returned in place of a successful result.
 *
 * @NOTES:
 * Provide any additional information that would be useful knowledge for future developers and reference.
*/


Allow User Abort [ On ]
Set Error Capture [ On ]
# 
# clear vars
Set Variable [ $OPENAI_API_KEY ; Value: "" ] 
Set Variable [ $api_end_point ; Value: "" ] 
Set Variable [ $prompt ; Value: "" ] 
Set Variable [ $aiprep ; Value: "" ] 
Set Variable [ $json ; Value: "" ] 
# 
# set vars
Set Variable [ $OPENAI_API_KEY ; Value: AIwork::OPENAI_API_KEY ] 
Set Variable [ $api_end_point ; Value: AIwork::api_end_point ] 
Set Variable [ $prompt ; Value: AIwork::ai_prompt & " " & AIwork::ai_custom_instructions_reponse // first attempt below but realized using the set json, it esc the quotes and makes nice all things and embarrassingly enough the calc didn't work anyway. Should of ask AI first. - Brian /… ] 
Set Variable [ $aiprep ; Value: JSONFormatElements ( JSONSetElement ( "";   [ "role"; "user"; 1 ];   [ "content"; $prompt; 1]  ) ) ] 
Set Variable [ $json ; Value: JSONSetElement ( "{}" ; 	[ "model" ; "gpt-4" ; JSONString ] ; 	[ "messages" ; "[" & $aiprep & "]" ; JSONArray ] ) ] 
# 
# function
Commit Records/Requests [ With dialog: Off ] 
# 
If [ IsEmpty ( $OPENAI_API_KEY ) ] 
	Beep
	Show Custom Dialog [ "ERROR" ; "Missing API Key. " ] 
	Exit Script [ Text Result: $error ] 
End If
# 
Set Field [ AIwork::ai_response_raw ; "" // Sets up for initial value and reset when run again. ] 
Set Field [ AIwork::ai_message ; "" // this will be the extracted 'message' from the json within the ai response raw. ] 
# 
Insert from URL [ Select ; With dialog: Off ; Target: AIwork::ai_response_raw ; $api_end_point ; cURL options: " --header \"Content-Type: application/json\" --header \"Authorization: Bearer " & $OPENAI_API_KEY & "\" --data @$json " ] 
# 
# put result in a field
# 
Set Field [ AIwork::ai_message ; Let ( [   d22 = JSONListValues ( AIwork::ai_response_raw; "choices" ) ; d23 = JSONListValues ( d22; "message" ) ] ; d23 ) ] 
# 
Go to Object [ Object Name: "result" ] 
# 
# hard coded fix of OpenAI response removing the last word "assistant" if present.
Set Field [ AIwork::ai_message ; Let ( [ text = AIwork::ai_message; lastWord = RightWords ( text; 1 ) ] ; If ( lastWord = "assistant" ; Left ( text ; Length ( text ) - Length ( lastWord ) ) ; text ) ) ] 
# 
Commit Records/Requests [ With dialog: Off ] 
# 
Exit Script [ Text Result: $null ] 
```

### ddr_this_layout_fields 
```
# #DB: AIwork_v2
# #LN: AIwork
# #TO: AIwork
# 
# 10 Regular Fields found on layout.
# 
# FieldComment
# ai_custom_instructions_reponse        How do you want the AI to answer?
# ai_message                            This is the core, important response extracted from the raw response.
# ai_prompt                             What do you want of AI?
# ai_response_raw                       The raw response from AI.
# api_end_point                         Where to send this cURL?
# CreationTimestamp                     Date and time each record was created
# evaluate                              This will evaluate the code returned from AI automatically to prove the result.
# ModificationTimestamp                 Date and time each record was last modified
# OPENAI_API_KEY                        API key for request.
# time_to_task                          This is the time it took to work this out.
# 
# FieldType
# ai_custom_instructions_reponse        Standard        Text       Indexed          1
# ai_message                            Standard        Text       Indexed          1
# ai_prompt                             Standard        Text       Indexed          1
# ai_response_raw                       Standard        Text       Unindexed        1
# api_end_point                         Standard        Text       Indexed          1
# CreationTimestamp                     Standard        Timestamp  Unindexed        1
# evaluate                              Standard        Text       Indexed          1
# ModificationTimestamp                 Standard        Timestamp  Unindexed        1
# OPENAI_API_KEY                        Standard        Text       Indexed          1
# time_to_task                          Standard        Time       Unindexed        1
# 
# FieldStyle
# ai_custom_instructions_reponse        Standard        
# ai_message                            Scrolling       
# ai_prompt                             Scrolling       
# ai_response_raw                       Scrolling       
# api_end_point                         Standard        
# CreationTimestamp                     Standard        
# evaluate                              Scrolling       
# ModificationTimestamp                 Standard        
# OPENAI_API_KEY                        Concealed       
# time_to_task                          Standard        
# 
# FieldBounds                                l     t     r     b   deg
# ai_custom_instructions_reponse           334   597   976   678     0   642.w    81.h
# ai_message                               181   235  1151   844     0   970.w   609.h
# ai_prompt                                181   235   976   565     0   795.w   330.h
# ai_response_raw                         1389   376  2184   486     0   795.w   110.h
# api_end_point                            183   274   825   343     0   642.w    69.h
# CreationTimestamp                        174    35   415    66     0   241.w    31.h
# evaluate                                 179   888  1149   971     0   970.w    83.h
# ModificationTimestamp                    684    35   925    66     0   241.w    31.h
# OPENAI_API_KEY                           183   235   825   266     0   642.w    31.h
# time_to_task                             420   172   541   203     0   121.w    31.h
# 
# 
# __________________________________________________________ 
# FieldType function from Claris help as of version 19.5.4: 
# - The first value is either Standard, StoredCalc, Summary, UnstoredCalc, External(Secure), External(Open), or Global. 
# - The second value is the field type: text, number, date, time, timestamp, or container. 
# - The third value is Indexed or Unindexed. 
# - The fourth value is the maximum number of repetitions defined for the field (if the field isn’t defined as a repeating field, this value is 1). 
# 
# FieldStyle function from Claris help as of version 19.5.4: 
# If the field has a value list associated with it, the FieldStyle function also returns the name of the value list. 
# - standard field returns Standard. 
# - standard field with - vertical scroll bar returns Scrolling. 
# - drop-down list returns Popuplist. 
# - pop-up menu returns Popupmenu. 
# - checkbox returns Checkbox. 
# - radio button returns RadioButton. 
# - drop-down calendar returns Calendar. 
# 
# FieldBounds function from Claris help as of version 19.5.4: 
# Returns the location, in points, of each field boundary and the field's rotation in degrees. 
# 
# How to use: 
#  Script is portable without dependencies. Copy into any solution and run.  Add shift key when running to clear resulting vars.  Clear the $expert var to suppress this extensive help text. It is under settings within the script. My work flow of investiation means I'll look at one to three or four layouts as that is the task at hand.  Yes, most can be found in the object inspector but this report is  something I wanted in my work life.  I can do this within a miniute so the $$report[XX] global var uses 60 repetition options, one per second and that is the XX part of the var.  It does add simple arithmetic to calc width and height from the FieldBounds fucntion which can help  isolate a possbile slight misalignment and is kind of fun to see. To see the report clearly (it uses spaces to show columns), in the Data Viewer,  "Add to Watch"  will move it to the Watch list and there you can open it and view it as intended. There you can use the built-in search too.  - All the best, Brian 
# 
# Report does not include <<Merge Fields>>.

```
## Cost to Operate

I started using GPT-4 heavily for this project on October 1st, 2023. I had been using it more casually since it first became available to me on December 11th, 2022, GPT-3 that is. So far this month, I've spent $5.77USD on top of the $20 monthly subscription fee. There's still one day left in October, so that number may change slightly.

Overall, it took some trial and error to figure out how to best use GPT-4 and prompt it in a way that provides helpful responses. But now that I've got the hang of it, I find the service incredibly useful. The additional costs on top of the base subscription have been worth it for me, considering how much time and effort GPT-4 has saved me. And I expect the value I get from it will only increase as I continue.
