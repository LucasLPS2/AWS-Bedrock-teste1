# AWS-Bedrock-teste1

{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Initialize",
  "QueryLanguage": "JSONata",
  "States": {
    "Initialize": {
      "Type": "Pass",
      "Next": "Passagem (1)",
      "Assign": {
        "counter": "{% $count($states.input.prompts) %}",
        "conversation_history": [
          ""
        ],
        "input_prompts": "{% $reverse($states.input.prompts) %}"
      }
    },
    "Passagem (1)": {
      "Type": "Pass",
      "Next": "Invoke model with prompt"
    },
    "Invoke model with prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/meta.llama3-3-70b-instruct-v1:0",
        "Body": {
          "prompt": "Estou progrmamando um jantar romantico, nesse jantar irei pedir um macarrão. Me dê 3 itens que combina com essa experiencia gastronômica",
          "temperature": 0,
          "top_p": 1,
          "max_gen_len": 1024
        },
        "ContentType": "application/json",
        "Accept": "*/*"
      },
      "Assign": {
        "conversation_history": "{% $append($conversation_history, $states.result.Body.generations[0].text) %}",
        "counter": "{% $counter - 1 %}"
      },
      "Next": "Passagem"
    },
    "Passagem": {
      "Type": "Pass",
      "Next": "Bedrock InvokeModel"
    },
    "Bedrock InvokeModel": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/meta.llama3-3-70b-instruct-v1:0",
        "Body": {
          "prompt": "Me mostre 3 bebidas mais adequado para o jantar romantico, levando em consideração, que foi escolhido o macarrão",
          "temperature": 0,
          "top_p": 1,
          "max_gen_len": 1024
        }
      },
      "Next": "Success"
    },
    "Success": {
      "Type": "Succeed",
      "Output": "{% $join($conversation_history[[1..$count($conversation_history)]], '.') %}"
    }
  }
}
