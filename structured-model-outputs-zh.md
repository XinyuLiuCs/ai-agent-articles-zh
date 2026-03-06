# 结构化模型输出

*Structured model outputs*---
[https://platform.openai.com/docs/guides/structured-outputs](https://platform.openai.com/docs/guides/structured-outputs)

JSON 是全球应用最广泛的数据交换格式之一。

JSON is one of the most widely used formats in the world for applications to exchange data.

结构化输出（Structured Outputs）是一项功能，它确保模型始终生成符合你提供的 [JSON Schema](https://json-schema.org/overview/what-is-jsonschema) 的响应，因此你无需担心模型遗漏必填字段或生成无效的枚举值。

Structured Outputs is a feature that ensures the model will always generate responses that adhere to your supplied [JSON Schema](https://json-schema.org/overview/what-is-jsonschema), so you don't need to worry about the model omitting a required key, or hallucinating an invalid enum value.

结构化输出的优势包括：

Some benefits of Structured Outputs include:

1. **可靠的类型安全**：无需验证或重试格式不正确的响应
2. **显式拒绝**：基于安全的模型拒绝现在可以通过编程方式检测
3. **更简单的提示**：无需使用强制性措辞的提示来实现一致的格式化
4. **Reliable type-safety:** No need to validate or retry incorrectly formatted responses
5. **Explicit refusals:** Safety-based model refusals are now programmatically detectable
6. **Simpler prompting:** No need for strongly worded prompts to achieve consistent formatting

除了在 REST API 中支持 JSON Schema 之外，OpenAI 的 [Python](https://github.com/openai/openai-python/blob/main/helpers.md#structured-outputs-parsing-helpers) 和 [JavaScript](https://github.com/openai/openai-node/blob/master/helpers.md#structured-outputs-parsing-helpers) SDK 还分别通过 [Pydantic](https://docs.pydantic.dev/latest/) 和 [Zod](https://zod.dev/) 提供了便捷的对象模式定义方式。下面展示了如何从非结构化文本中提取符合代码中定义的模式的信息。

In addition to supporting JSON Schema in the REST API, the OpenAI SDKs for [Python](https://github.com/openai/openai-python/blob/main/helpers.md#structured-outputs-parsing-helpers) and [JavaScript](https://github.com/openai/openai-node/blob/master/helpers.md#structured-outputs-parsing-helpers) also make it easy to define object schemas using [Pydantic](https://docs.pydantic.dev/latest/) and [Zod](https://zod.dev/) respectively. Below, you can see how to extract information from unstructured text that conforms to a schema defined in code.

### 支持的模型

### Supported models

结构化输出可在我们[最新的大语言模型](https://developers.openai.com/api/docs/models)中使用，从 GPT-4o 开始。较旧的模型如 `gpt-4-turbo` 及更早版本可使用 [JSON 模式](#json-模式)作为替代。

Structured Outputs is available in our [latest large language models](https://developers.openai.com/api/docs/models), starting with GPT-4o. Older models like `gpt-4-turbo` and earlier may use [JSON mode](#json-mode) instead.

## 何时使用函数调用的结构化输出 vs `text.format` 的结构化输出

## When to use Structured Outputs via function calling vs via `text.format`

结构化输出在 OpenAI API 中有两种使用方式：

Structured Outputs is available in two forms in the OpenAI API:

1. 通过[函数调用](https://developers.openai.com/api/docs/guides/function-calling)使用
2. 通过 `json_schema` 响应格式使用
3. When using [function calling](https://developers.openai.com/api/docs/guides/function-calling)
4. When using a `json_schema` response format

当你构建的应用需要将模型与应用功能桥接时，函数调用非常有用。

Function calling is useful when you are building an application that bridges the models and functionality of your application.

例如，你可以让模型访问查询数据库的函数来构建 AI 助手帮助用户处理订单，或者使用与 UI 交互的函数。

For example, you can give the model access to functions that query a database in order to build an AI assistant that can help users with their orders, or functions that can interact with the UI.

相反，通过 `response_format` 使用结构化输出更适用于你希望在模型回复用户时指定结构化模式的场景，而非模型调用工具时。

Conversely, Structured Outputs via `response_format` are more suitable when you want to indicate a structured schema for use when the model responds to the user, rather than when the model calls a tool.

例如，如果你正在构建一个数学辅导应用，你可能希望助手使用特定的 JSON Schema 回复用户，以便你可以生成一个将模型输出的不同部分以不同方式展示的 UI。

For example, if you are building a math tutoring application, you might want the assistant to respond to your user using a specific JSON Schema so that you can generate a UI that displays different parts of the model's output in distinct ways.

简单来说：

Put simply:

- 如果你要将模型连接到系统中的工具、函数、数据等，应使用函数调用
- 如果你想在模型回复用户时结构化模型的输出，应使用结构化的 `text.format`
- If you are connecting the model to tools, functions, data, etc. in your system, then you should use function calling
- If you want to structure the model's output when it responds to the user, then you should use a structured `text.format`

本指南的其余部分将重点介绍 Responses API 中非函数调用的使用场景。要了解如何在函数调用中使用结构化输出，请查阅[函数调用](https://developers.openai.com/api/docs/guides/function-calling#function-calling-with-structured-outputs)指南。

The remainder of this guide will focus on non-function calling use cases in the Responses API. To learn more about how to use Structured Outputs with function calling, check out the [Function Calling](https://developers.openai.com/api/docs/guides/function-calling#function-calling-with-structured-outputs) guide.

### 结构化输出 vs JSON 模式

### Structured Outputs vs JSON mode

结构化输出是 [JSON 模式](#json-模式)的进化版本。虽然两者都能确保生成有效的 JSON，但只有结构化输出能保证输出符合模式定义。结构化输出和 JSON 模式在 Responses API、Chat Completions API、Assistants API、Fine-tuning API 和 Batch API 中均受支持。

Structured Outputs is the evolution of [JSON mode](#json-mode). While both ensure valid JSON is produced, only Structured Outputs ensure schema adherence. Both Structured Outputs and JSON mode are supported in the Responses API, Chat Completions API, Assistants API, Fine-tuning API and Batch API.

我们建议在可能的情况下始终使用结构化输出而非 JSON 模式。

We recommend always using Structured Outputs instead of JSON mode when possible.

但需注意，使用 `response_format: {type: "json_schema", ...}` 的结构化输出仅支持 `gpt-4o-mini`、`gpt-4o-mini-2024-07-18` 和 `gpt-4o-2024-08-06` 及之后的模型快照。

However, Structured Outputs with `response_format: {type: "json_schema", ...}` is only supported with the `gpt-4o-mini`, `gpt-4o-mini-2024-07-18`, and `gpt-4o-2024-08-06` model snapshots and later.


|               | 结构化输出 (Structured Outputs)                                                 | JSON 模式 (JSON Mode)                         |
| ------------- | -------------------------------------------------------------------------- | ------------------------------------------- |
| **输出有效 JSON** | 是                                                                          | 是                                           |
| **符合模式定义**    | 是（参见[支持的模式](#支持的模式)）                                                       | 否                                           |
| **兼容模型**      | `gpt-4o-mini`、`gpt-4o-2024-08-06` 及之后版本                                    | `gpt-3.5-turbo`、`gpt-4-`* 和 `gpt-4o-*` 模型   |
| **启用方式**      | `text: { format: { type: "json_schema", "strict": true, "schema": ... } }` | `text: { format: { type: "json_object" } }` |



|                        | Structured Outputs                                                         | JSON Mode                                        |
| ---------------------- | -------------------------------------------------------------------------- | ------------------------------------------------ |
| **Outputs valid JSON** | Yes                                                                        | Yes                                              |
| **Adheres to schema**  | Yes (see [supported schemas](#supported-schemas))                          | No                                               |
| **Compatible models**  | `gpt-4o-mini`, `gpt-4o-2024-08-06`, and later                              | `gpt-3.5-turbo`, `gpt-4-`* and `gpt-4o-*` models |
| **Enabling**           | `text: { format: { type: "json_schema", "strict": true, "schema": ... } }` | `text: { format: { type: "json_object" } }`      |


## 示例

## Examples

Chain of thought

Structured data extraction

UI generation

Moderation

## 如何通过 `text.format` 使用结构化输出

## How to use Structured Outputs with `text.format`

## 结构化输出中的拒绝处理

## Refusals with Structured Outputs

当将结构化输出与用户生成的输入配合使用时，OpenAI 模型可能偶尔会出于安全原因拒绝执行请求。由于拒绝响应不一定遵循你在 `response_format` 中提供的模式，API 响应将包含一个名为 `refusal` 的新字段，用于表示模型拒绝了该请求。

When using Structured Outputs with user-generated input, OpenAI models may occasionally refuse to fulfill the request for safety reasons. Since a refusal does not necessarily follow the schema you have supplied in `response_format`, the API response will include a new field called `refusal` to indicate that the model refused to fulfill the request.

当输出对象中出现 `refusal` 属性时，你可以在 UI 中展示拒绝信息，或在消费响应的代码中加入条件逻辑来处理请求被拒绝的情况。

When the `refusal` property appears in your output object, you might present the refusal in your UI, or include conditional logic in code that consumes the response to handle the case of a refused request.

拒绝响应的 API 返回大致如下：

The API response from a refusal will look something like this:

## 技巧与最佳实践

## Tips and best practices

#### 处理用户生成的输入

#### Handling user-generated input

如果你的应用使用用户生成的输入，请确保提示中包含处理输入无法生成有效响应的情况的说明。

If your application is using user-generated input, make sure your prompt includes instructions on how to handle situations where the input cannot result in a valid response.

模型会始终尝试遵循提供的模式，如果输入与模式完全无关，可能会导致幻觉。

The model will always try to adhere to the provided schema, which can result in hallucinations if the input is completely unrelated to the schema.

你可以在提示中指定：当模型检测到输入与任务不兼容时，返回空参数或特定的提示语句。

You could include language in your prompt to specify that you want to return empty parameters, or a specific sentence, if the model detects that the input is incompatible with the task.

#### 处理错误

#### Handling mistakes

结构化输出仍然可能包含错误。如果你发现了错误，请尝试调整指令、在系统指令中提供示例，或将任务拆分为更简单的子任务。有关如何调整输入的更多指导，请参阅[提示工程指南](https://developers.openai.com/api/docs/guides/prompt-engineering)。

Structured Outputs can still contain mistakes. If you see mistakes, try adjusting your instructions, providing examples in the system instructions, or splitting tasks into simpler subtasks. Refer to the [prompt engineering guide](https://developers.openai.com/api/docs/guides/prompt-engineering) for more guidance on how to tweak your inputs.

#### 避免 JSON Schema 与类型定义的不一致

#### Avoid JSON schema divergence

为防止 JSON Schema 和编程语言中对应的类型定义出现不一致，我们强烈建议使用原生的 Pydantic/Zod SDK 支持。

To prevent your JSON Schema and corresponding types in your programming language from diverging, we strongly recommend using the native Pydantic/zod sdk support.

如果你更倾向于直接指定 JSON Schema，可以添加 CI 规则来标记 JSON Schema 或底层数据对象被修改的情况，或添加 CI 步骤从类型定义自动生成 JSON Schema（或反之）。

If you prefer to specify the JSON schema directly, you could add CI rules that flag when either the JSON schema or underlying data objects are edited, or add a CI step that auto-generates the JSON Schema from type definitions (or vice-versa).

## 流式传输

## Streaming

## 支持的模式

## Supported schemas

## JSON 模式

## JSON mode

JSON 模式是结构化输出功能的一个更基础的版本。虽然 JSON 模式确保模型输出为有效的 JSON，但结构化输出能可靠地使模型输出与你指定的模式匹配。如果你的使用场景支持结构化输出，我们建议优先使用它。

JSON mode is a more basic version of the Structured Outputs feature. While JSON mode ensures that model output is valid JSON, Structured Outputs reliably matches the model's output to the schema you specify. We recommend you use Structured Outputs if it is supported for your use case.

启用 JSON 模式后，模型的输出将确保为有效的 JSON，但在某些边缘情况下可能出现例外，你应当妥善检测和处理这些情况。

When JSON mode is turned on, the model's output is ensured to be valid JSON, except for in some edge cases that you should detect and handle appropriately.

要在 Responses API 中启用 JSON 模式，可以将 `text.format` 设置为 `{ "type": "json_object" }`。如果你使用的是函数调用，JSON 模式会自动开启。

To turn on JSON mode with the Responses API you can set the `text.format` to `{ "type": "json_object" }`. If you are using function calling, JSON mode is always turned on.

重要注意事项：

Important notes:

- 使用 JSON 模式时，你必须始终通过对话中的某条消息（例如系统消息）指示模型生成 JSON。如果没有包含明确的生成 JSON 的指令，模型可能会生成无尽的空白字符，请求将持续运行直到达到 token 上限。为帮助你避免遗忘，如果上下文中没有出现"JSON"字符串，API 将抛出错误。
- JSON 模式不会保证输出匹配任何特定的模式，只保证输出是有效的且可以无错误地解析。你应该使用结构化输出来确保输出匹配你的模式，如果无法使用结构化输出，则应使用验证库并可能配合重试来确保输出符合你期望的模式。
- 你的应用必须检测和处理可能导致模型输出不是完整 JSON 对象的边缘情况（参见下文）。
- When using JSON mode, you must always instruct the model to produce JSON via some message in the conversation, for example via your system message. If you don't include an explicit instruction to generate JSON, the model may generate an unending stream of whitespace and the request may run continually until it reaches the token limit. To help ensure you don't forget, the API will throw an error if the string "JSON" does not appear somewhere in the context.
- JSON mode will not guarantee the output matches any specific schema, only that it is valid and parses without errors. You should use Structured Outputs to ensure it matches your schema, or if that is not possible, you should use a validation library and potentially retries to ensure that the output matches your desired schema.
- Your application must detect and handle the edge cases that can result in the model output not being a complete JSON object (see below)

## 处理边缘情况

## Handling edge cases

## 资源

## Resources

要了解更多关于结构化输出的信息，我们建议浏览以下资源：

To learn more about Structured Outputs, we recommend browsing the following resources:

- 查阅我们关于结构化输出的[入门教程](https://developers.openai.com/cookbook/examples/structured_outputs_intro)
- 学习[如何使用结构化输出构建多智能体系统](https://developers.openai.com/cookbook/examples/structured_outputs_multi_agent)
- Check out our [introductory cookbook](https://developers.openai.com/cookbook/examples/structured_outputs_intro) on Structured Outputs
- Learn [how to build multi-agent systems](https://developers.openai.com/cookbook/examples/structured_outputs_multi_agent) with Structured Outputs

