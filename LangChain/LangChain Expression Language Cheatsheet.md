This is a quick reference for all the most important LCEL primitives. For more advanced usage see the [LCEL how-to guides](https://python.langchain.com/v0.2/docs/how_to/#langchain-expression-language-lcel) and the [full API reference](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html).

### Invoke a runnable
#### [Runnable.invoke()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.invoke) / [Runnable.ainvoke()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.ainvoke)
```python
form langchain_core.runnables import RunnableLambda

runnable = RunnableLambda(lambda x: str(x))
runnable.invoke(5) # '5'

# Async variant:
# await runnable.ainvoke(5) # '5'
```
**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)

### Batch a runnable

#### [Runnable.batch()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.batch) / [Runnable.abatch()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.abatch)

```python
from langchain_core.runnables import RunnableLambda
runnable = RunnableLambda(lambda x: str(x))
runnable.batch([7, 8, 9])
# Async variant:
# await runnable.abatch([7, 8, 9])
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)
```console
['7', '8', '9']
```

### Stream a runnable

#### [Runnable.stream()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.stream) / [Runnable.astream()](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.astream)

```python
from langchain_core.runnables import RunnableLambda

def func(x):
    for y in x:
        yield str(y)

runnable = RunnableLambda(func)

for chunk in runnable.stream(range(5)):
    print(chunk)

# Async variant:
# async for chunk in await runnable.astream(range(5)):
#     print(chunk)

```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)

```console
0
1
2
3
4
```

### Compose runnables

#### Pipe operator `|`

>[!note] Notes:
>- each runnable in the chain is called in the same order they are composed
>- first runnable is called with input provided by `invoke` or `batch` etc.
>- output of the previous runnable is as the input to the next runnable in the chain

The main composition primitives are `RunnableSequence` and `RunnableParallel`.

`RunnableSequence` invokes a series of runnables sequentially, with one runnable’s output serving as the next's input. Construct using the `|` operator or by passing a list of runnables to RunnableSequence.

`RunnableParallel` invokes runnables concurrently, providing the same input to each. Construct it using a dict literal within a sequence or by passing a dict to RunnableParallel.

```python
from langchain_core.runnables import RunnableLambda

# A RunnableSequence constructed using the `|` operator
sequence = RunnableLambda(lambda x: x + 1) | RunnableLambda(lambda x: x * 2)
sequence.invoke(1) # 4
sequence.batch([1, 2, 3]) # [4, 6, 8]

# A sequence that contains a RunnableParallel constructed using a dict literal
sequence = RunnableLambda(lambda x: x + 1) | {
    'mul_2': RunnableLambda(lambda x: x * 2),
    'mul_5': RunnableLambda(lambda x: x * 5)
}
sequence.invoke(1) # {'mul_2': 4, 'mul_5': 10}
```

```python
from langchain_core.runnables import RunnableLambda

runnable1 = RunnableLambda(lambda x: int(x))
runnable2 = RunnableLambda(lambda x: x ** 2)

chain = runnable1 | runnable2

chain.batch(['1', '2', '3', '4'])
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)

```console
[1, 4, 9, 16]
```

### Turn any function into a runnable

#### [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)

```python
from langchain_core.runnables import RunnableLambda

def func(x):
    return x + 5

runnable = RunnableLambda(func)
runnable.invoke(2)
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html)

```console
7
```

### Merge input and output dicts

#### [RunnablePassthrough.assign](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html)

Runnable to *passthrough* the inputs unchanged or with additional keys.

This runnable behaves almost like the identity function, except that it can be configured to add additional keys to the output, if the input is a dict.

The examples below demonstrate this Runnable works using a few simple chains. The chains rely on simple lambdas to make the examples easy to execute and experiment with.

**Examples**

```python
from langchain_core.runnables import (
    RunnableLambda,
    RunnableParallel,
    RunnablePassthrough,
)

runnable = RunnableParallel(
    origin=RunnablePassthrough(),
    modified=lambda x: x+1
)

runnable.invoke(1) # {'origin': 1, 'modified': 2}

def fake_llm(prompt: str) -> str: # Fake LLM for the example
    return "completion"

chain = RunnableLambda(fake_llm) | {
    'original': RunnablePassthrough(), # Original LLM output
    'parsed': lambda text: text[::-1] # Parsing logic
}

chain.invoke('hello') # {'original': 'completion', 'parsed': 'noitelpmoc'}
```

In some cases, it may be useful to pass the input through while adding some keys to the output. In this case, you can use the assign method:

```python
from langchain_core.runnables import RunnablePassthrough

def fake_llm(prompt: str) -> str: # Fake LLM for the example
    return "completion"

runnable = {
    'llm1':  fake_llm,
    'llm2':  fake_llm,
} | RunnablePassthrough.assign(
    total_chars=lambda inputs: len(inputs['llm1'] + inputs['llm2'])
)

runnable.invoke('hello')
# {'llm1': 'completion', 'llm2': 'completion', 'total_chars': 20}
```

```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough

runnable1 = RunnableLambda(lambda x: x["foo"] + 7)

chain = RunnablePassthrough.assign(bar=runnable1)

chain.invoke({"foo": 10})
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html) | [RunnablePassthrough](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.passthrough.RunnablePassthrough.html)

```console
{'foo': 10, 'bar': 17}
```

> [!note] Notes:
> _class_ langchain_core.runnables.config.**RunnableConfig**
> 
> Configuration for a Runnable.
> 
> **tags**_: List[str]_[](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html#langchain_core.runnables.config.RunnableConfig.tags "Permalink to this definition")
> Tags for this call and any sub-calls (eg. a Chain calling an LLM). You can use these to filter calls.
> 
> **metadata**_: Dict[str, Any]_[](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html#langchain_core.runnables.config.RunnableConfig.metadata "Permalink to this definition")
> Metadata for this call and any sub-calls (eg. a Chain calling an LLM). Keys should be strings, values should be JSON-serializable.
> 
> **callbacks**: Optional[Union[List, Any]]
> Callbacks for this call and any sub-calls (eg. a Chain calling an LLM). Tags are passed to all callbacks, metadata is passed to handle*Start callbacks.
> 
> **run_name**: str
> Name for the tracer run for this call. Defaults to the name of the class.
> 
> **max_concurrency**: Optional[int]
> Maximum number of parallel calls to make. If not provided, defaults to ThreadPoolExecutor's default.
> 
> **recursion_limit**: int
> Maximum number of times a call can recurse. If not provided, defaults to 25.
> 
> **configurable**: Dict[str, Any]
> Runtime values for attributes previously made configurable on this Runnable, or sub-Runnables, through .configurable_fields() or .configurable_alternatives(). Check .output_schema() for a description of the attributes that have been made configurable.
> 
> **run_id**: Optional[UUID]
> Unique identifier for the tracer run for this call. If not provided, a new UUID will be generated.

### Configure runnable execution

#### [RunnableConfig](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html)

```python
from langchain_core.runnables import RunnableLambda, RunnableParallel

runnable1 = RunnableLambda(lambda x: {"foo": x})
runnable2 = RunnableLambda(lambda x: [x] * 2)
runnable3 = RunnableLambda(lambda x: str(x))

chain = RunnableParallel(first=runnable1, second=runnable2, third=runnable3)

chain.invoke(7, config={"max_concurrency": 2})
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html) | [RunnableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)

```console
{'first': {'foo': 7}, 'second': [7, 7], 'third': '7'}
```

### Add default config to runnable

#### [Runnable.with_config](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#langchain_core.runnables.base.Runnable.with_config)

```python
from langchain_core.runnables import RunnableLambda, RunnableParallel

runnable1 = RunnableLambda(lambda x: {"foo": x})
runnable2 = RunnableLambda(lambda x: [x] * 2)
runnable3 = RunnableLambda(lambda x: str(x))

chain = RunnableParallel(first=runnable1, second=runnable2, third=runnable3)
configured_chain = chain.with_config(max_concurrency=2)

chain.invoke(7)
```

**API Reference:**[RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html) | [RunnableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)

```console
{'first': {'foo': 7}, 'second': [7, 7], 'third': '7'}
```

### Make runnable attributes configurable

#### [Runnable.with_configurable_fields](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSerializable.html#langchain_core.runnables.base.RunnableSerializable.configurable_fields)

```python
from typing import Any, Optional

from langchain_core.runnables import (
    ConfigurableField,
    RunnableConfig,
    RunnableSerializable,
)

class FooRunnable(RunnableSerializable[dict, dict]):
    output_key: str

    def invoke(
        self, input: Any, config: Optional[RunnableConfig] = None, **kwargs: Any
    ) -> list:
        return self._call_with_config(self.subtract_seven, input, config, **kwargs)

    def subtract_seven(self, input: dict) -> dict:
        return {self.output_key: input["foo"] - 7}

runnable1 = FooRunnable(output_key="bar")
configurable_runnable1 = runnable1.configurable_fields(
    output_key=ConfigurableField(id="output_key")
)

configurable_runnable1.invoke(
    {"foo": 10}, config={"configurable": {"output_key": "not bar"}}
)
```

**API Reference:**[ConfigurableField](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.utils.ConfigurableField.html) | [RunnableConfig](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html) | [RunnableSerializable](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSerializable.html)

```console
{'not bar': 3}
```

```console
configurable_runnable1.invoke({"foo": 10})
```

```console
{'bar': 3}
```

### Make chain components configurable

#### [Runnable.with_configurable_alternatives](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableSerializable.html#langchain_core.runnables.base.RunnableSerializable.configurable_alternatives)

```python
llm = ChatAnthropic(temperature=0).configurable_alternatives(
    # This gives this field an id
    # When configuring the end runnable, we can then use this id to configure this field
    ConfigurableField(id="llm"),
    # This sets a default_key.
    # If we specify this key, the default LLM (ChatAnthropic initialized above) will be used
    default_key="anthropic",
    # This adds a new option, with name `openai` that is equal to `ChatOpenAI()`
    openai=ChatOpenAI(),
    # This adds a new option, with name `gpt4` that is equal to `ChatOpenAI(model="gpt-4")`
    gpt4=ChatOpenAI(model="gpt-4"),
    # You can add more configuration options here
)
prompt = PromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | llm
chain.invoke(7, config={"configurable": {"second_step": "string"}})
```

**API Reference:**[RunnableConfig](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.config.RunnableConfig.html) | [RunnableLambda](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableLambda.html) | [RunnableParallel](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.RunnableParallel.html)

```python
# By default it will call Anthropic
chain.invoke({"topic": "bears"})
```

```console
AIMessage(content=" Here's a silly joke about bears:\n\nWhat do you call a bear with no teeth?\nA gummy bear!")
```

```python
# We can use `.with_config(configurable={"llm": "openai"})` to specify an llm to use
chain.with_config(configurable={"llm": "openai"}).invoke({"topic": "bears"})
```

```console
AIMessage(content="Sure, here's a bear joke for you:\n\nWhy don't bears wear shoes?\n\nBecause they already have bear feet!")
```

```python
# If we use the `default_key` then it uses the default
chain.with_config(configurable={"llm": "anthropic"}).invoke({"topic": "bears"})
```

```console
AIMessage(content=" Here's a silly joke about bears:\n\nWhat do you call a bear with no teeth?\nA gummy bear!")
```

