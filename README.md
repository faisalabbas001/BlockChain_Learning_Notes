





input guradrails 




# from agents import Agent, Runner, AsyncOpenAI, OpenAIChatCompletionsModel, set_tracing_disabled,function_tool,RunContextWrapper
# from pydantic import BaseModel
# from typing import Any
# import asyncio
# from openai.types.responses import ResponseTextDeltaEvent
# from agents import input_guardrail,GuardrailFunctionOutput,TResponseInputItem
# set_tracing_disabled(disabled=True)

# # 1. Setup provider and model
# provider = AsyncOpenAI(
#     api_key="AIzaSyApe0wF7oJz4UPY9Wf7KhWc4V8Isf_dvQM", 
#     base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
# )

# model = OpenAIChatCompletionsModel(openai_client=provider, model="gemini-2.0-flash-exp")


# class OutputPython(BaseModel):
#     is_python_code:bool
#     reasoning:str
 
# input_guadrail_agent=Agent(
#     name="Input Guadrail Agent",
#     model=model,
#     instructions="if the user has been asked to write a python code, then you should return the python code and the reasoning for the code and when the user has been ask the not python code  you should return the reasoning for the code and the python code should be false",
#     output_type=OutputPython
# )


# @input_guardrail
# async def input_guardrails_func(
#     context:RunContextWrapper,agent:Agent,input:str | list[ResponseTextDeltaEvent]
# )->GuardrailFunctionOutput:
#     result=await Runner.run(input_guadrail_agent,input)


#     return GuardrailFunctionOutput(
#     output_info=result.final_output,
#     tripwire_triggered=not result.final_output.is_python_code
#     )



# main_agent=Agent(
#     name="Main Agent",
#     model=model,
#     instructions="you are a helpful assistant that can answer questions and help with tasks",
#     input_guardrails=[input_guardrails_func]
# )


# try:
#     response=Runner.run_sync(main_agent,input="write a python  code to print hello world")
#     print(response.final_output)
# except Exception as e:
#     print(e)









from agents import Agent,Runner,OpenAIChatCompletionsModel,set_tracing_disabled,AsyncOpenAI
from pydantic import BaseModel
from agents import input_guardrail,GuardrailFunctionOutput,RunContextWrapper,TResponseInputItem
set_tracing_disabled(disabled=True)

provider=AsyncOpenAI(
    api_key="AIzaSyApe0wF7oJz4UPY9Wf7KhWc4V8Isf_dvQM",
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
)

model=OpenAIChatCompletionsModel(openai_client=provider,model="gemini-2.0-flash-exp")


class OutputResponse(BaseModel):
   is_cricket_related:bool
   reasoning:str


checker_input_security=Agent(
    name="input guadrials ",
    instructions="you give the answer only for cricket related questions",
    model=model,
    output_type=OutputResponse
)

@input_guardrail
async def input_guardrails_func(
    context:RunContextWrapper,agent:Agent,input:str | list[TResponseInputItem]
)->GuardrailFunctionOutput:
    result=await Runner.run(checker_input_security,input)
    return GuardrailFunctionOutput(
        output_info=result.final_output,
        tripwire_triggered=not result.final_output.is_cricket_related
    )


main_agent=Agent(
    name="cricket agent",
    model=model,
    instructions="you are the helpfull for cricket game ",
    input_guardrails=[input_guardrails_func]
)

try:
    response=Runner.run_sync(main_agent,input="how to play the volly ball")
    print(response.final_output)
except Exception as e:
    print(e)
