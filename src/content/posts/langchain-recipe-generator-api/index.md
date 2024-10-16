---
title: Recipe API using LangChain, Ollama and FastApi
published: 2024-07-09
description: ''
image: ''
tags: ['langchain', 'python', 'fastapi', 'Ollama']
category: 'DEV'
draft: false 
---

In the blog we will see how to use FastAPI and LangChain to create a recipe generator.

## **Prerequisites**

Before we dive into the code, make sure you have the following installed:

1.  Python ([https://www.python.org/](https://www.python.org/))
2.  Ollama ([https://ollama.com/](https://ollama.com/))
3.  Install necessary Python Libraries

```python
pip install langchain fastapi uvicorn pydantic langchain-community
```

## Step-by-Step Code Explanation

**Step 1:** Import Necessary Libraries

```python
from fastapi import FastAPI, HTTPException, Body
from pydantic import BaseModel, Field
from langchain_community.llms import ollama
from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import JsonOutputParser
```

**Step 2:** Define the FastAPI Application

```python
app = FastAPI()
```

This line initializes a FastAPI application instance.

**Step 3:** Define the Ollama Model

```python
# Define the Ollama model
model = ollama.Ollama(
    model='dolphin-llama3',#llama3, mistral, etc.
)
```

We define the Ollama model that we’ll be using for recipe generation. It can be any model that is in the llama directory.

**Step 4:** Create the Pydantic Model for Recipes

```python
class Recipe(BaseModel):
    name: str = Field(description="The name of the recipe")
    ingredients: list[str] = Field(description="The list of ingredients for the recipe with quantity and unit")
    instructions: list[str] = Field(description="The list of instructions to make the recipe")
    cooking_time: str = Field(description="The cooking time for the recipe")
    serving_size: str = Field(description="The serving size for the recipe")
```

We define a `Recipe` model using Pydantic. This model includes fields for the recipe name, ingredients, instructions, cooking time, and serving size, with descriptions for each field.

**Step 5:** Set Up the JSON Output Parser

```python
parser = JsonOutputParser(pydantic_object=Recipe)
```

We set up a parser using LangChain’s `JsonOutputParser` to parse the output from the language model into our `Recipe` Pydantic model.

**Step 6:** Create the Prompt Template

```python
prompt = PromptTemplate(
    template="You are a master chef. Create a detailed recipe using the following ingredients and give the answers and follow the instructions no extras\n{format_instructions}\n ingredients: {ingredients}\n cuisine_type: {cuisine_type}\n",    
    input_variables=["ingredients", "cuisine_type"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)
```

We create a `PromptTemplate` to generate the prompt for the language model. The template includes placeholders for ingredients and cuisine type, along with instructions for the format.

**Step 7:** Define the Recipe Generation Endpoint

```python
@app.post('/generate_recipe', response_model=Recipe)
def generate_recipe(ingredients: str = Body(...), cuisine_type: str = Body(...)):
    print(type(ingredients), cuisine_type)
    try:
        # Invoke the chain
        chain = prompt | model | parser
        response = chain.invoke({"ingredients": ingredients, "cuisine_type": cuisine_type})
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to generate recipe: {str(e)}")
```

We define a POST endpoint `/generate_recipe` that takes `ingredients` and `cuisine_type` as input. It invokes the LangChain pipeline (consisting of the prompt, model, and parser) and returns the generated recipe. In case of any exceptions, it raises an HTTP 500 error.

**Step 8:** Run the Application

```python
if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='127.0.0.1', port=8000)
```

Finally, we run the FastAPI application using Uvicorn on `localhost` at port `8000`.

## Full Code:

```python
from fastapi import FastAPI, HTTPException, Body
from pydantic import BaseModel, Field
from langchain_community.llms import ollama
from langchain.prompts import PromptTemplate
from langchain_core.output_parsers import JsonOutputParser

app = FastAPI()

# Define the Ollama model
model = ollama.Ollama(
    model='dolphin-llama3',
)

# Define the Pydantic model for the Recipe
class Recipe(BaseModel):
    name: str = Field(description="The name of the recipe")
    ingredients: list[str] = Field(description="The list of ingredients for the recipe with quantity and unit")
    instructions: list[str] = Field(description="The list of instructions to make the recipe")
    cooking_time: str = Field(description="The cooking time for the recipe")
    serving_size: str = Field(description="The serving size for the recipe")

# Set up a parser + inject instructions into the prompt template.
parser = JsonOutputParser(pydantic_object=Recipe)

# Set up the prompt template
prompt = PromptTemplate(
    template="You are a master chef. Create a detailed recipe using the following ingredients and give the answers and follow the instructions no extras\n{format_instructions}\n ingredients: {ingredients}\n cuisine_type: {cuisine_type}\n",    
    input_variables=["ingredients", "cuisine_type"],
    partial_variables={"format_instructions": parser.get_format_instructions()},
)

@app.post('/generate_recipe', response_model=Recipe)
def generate_recipe(ingredients: str = Body(...), cuisine_type: str = Body(...)):
    print(type(ingredients), cuisine_type)
    try:
        # Invoke the chain
        chain = prompt | model | parser
        response = chain.invoke({"ingredients": ingredients, "cuisine_type": cuisine_type})
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to generate recipe: {str(e)}")

if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='127.0.0.1', port=8000)
```

## Conclusion

This tutorial walked you through setting up a recipe generator using FastAPI and LangChain. By defining a FastAPI application, a Pydantic model for recipes, and utilizing LangChain for generating recipes, you can create a powerful and flexible API for recipe generation. Feel free to extend and customize this application to suit your specific needs. Happy cooking!