# Bulleted Notes Book Summaries

_Built With: Python 3.11.9_

## Introduction
This project creates bulleted notes summaries of books and other long texts, particularly epub and pdf which have ToC metadata available.

When the ebooks contain approrpiate metadata, we are able to easily automate the extraction of chapters from most books, and split them into ~2000 token chunks, with fallbacks in the case your document doesn't have that.

### Main Idea

The main idea of this project is that we don't want to talk to the entire document at once, but we split it into many small chunks and ask questions to those, for improved granularity of response. We don't want a one page summary of the book, we want a summary of each of the book's subsections. Furthermore, we can ask arbitrary questions to those parts. Asking the same question to every part of the text, rather than one question to the whole thing at once.

You can check the [depreciated walkthroughs and rankings](notes/depreciated/) for information on some of my learning process with LLM and how I came to certain decisions.

### Comparison with RAG

Similar to Retrieval Augmented Generation (RAG), we split the document into many parts, so they fit into the context. The difference is that RAG systems try to determine what is the best chunk to ask their question to. Instead, we ask the same questions to *every part of the document*.

Its very important towards unlocking the full capabilities of LLM without relying on a multitude of 3rd party apps.

## Contents
- [Setup](#setup)
  - [Python Environment](#python-environment)
  - [Install Dependencies](#install-dependencies)
  - [Download Models](#download-models)
  - [Change model names](#change-model-names)
  - [Update Config File `_config.yaml`](#update-config-file-_configyaml)
- [Usage](#usage)
  - [Convert E-book to chunked CSV or TXT](#convert-e-book-to-chunked-csv-or-txt)
  - [Generate Summary](#generate-summary)
- [Models](#models)
  - [Ollama](#ollama)
  - [HuggingFace](#huggingface)
- [Check your eBook for clickable ToC](#check-your-ebook-for-clickable-toc)
  - [Firefox](#firefox)
  - [Brave](#brave)
- [Disclaimer](#disclaimer)
- [Inspiration](#inspiration)

## Setup
### Python Environment

Before starting, ensure you have Python 3.11.9 installed. If not, you can use conda or pyenv to manage Python versions:

**Using conda:**
1. Install Miniconda from: https://docs.conda.io/en/latest/miniconda.html
2. Create a new environment: `conda create -n book_summary python=3.11.9`
3. Activate the environment: `conda activate book_summary`

**Using pyenv:**
1. Install pyenv: https://github.com/pyenv/pyenv#installation
2. Install Python 3.11.9: `pyenv install 3.11.9`
3. Set local version: `pyenv local 3.11.9`

### Install Dependencies
```
pip install -r requirements.txt
```
- [Install Ollama](https://github.com/ollama/ollama?tab=readme-ov-file#ollama)

### Download Models

#### 1. **Download a copy of Mistral Instruct v0.2 Bulleted Notes Fine-Tune**

`ollama pull cognitivetech/obook_summary:q5_k_m`

#### 2. **Set up a title model**

##### a) *Download a preconfigured model*

`ollama pull cognitivetech/obook_title:q3_k_m`

For your convenience Mistral 7b 0.3 is packaged with the necessary message history for title creation. 

***or***

##### b) *Append this* [message history](Modelfile) *to the Modelfile of your choice *

### Update Config File `_config.yaml`

Ensure the defaults are set accordingly 

```yaml
defaults:
  prompt: bnotes # default prompt
  summary: cognitivetech/obook_summary:q5_k_m   # default model for summary
  title: cognitivetech/obook_title:q3_k_m  # default model for title generation
prompts:
  bnotes: # Only this prompt should go into the summary fine-tune. 
    prompt: Write comprehensive bulleted notes summarizing the provided text, with
      headings and terms in bold.
  clean:  # Other prompts from here forward go into a general purpose model
    prompt: Repeat back this text exactly, remove only garbage characters that do
      not contribute to the flow of text. Output only the main text content, condensed
      onto a single line. If you encounter any chapter boundaries or subheadings,
      start a new line beginning with its title.
  concise:
    prompt: Repeat the provided passage, with Concision.
  md:
    prompt: 'Print these notes in proper markdown format, with headings marked as
      bold with double asterisks and terms in bold also, and bullet points as `-`.
      Print the notes exactly, word-for-word, do not elaborate, do not add headings
      with #'
  research:
    prompt: Does this text make any arguments? If so, list them here.
  sum:
    prompt: Comprehensive bulleted notes with headings and terms in bold.
  teacher:
    prompt: 'Write a list of questions that can be answered by 3rd graders who are
      reading the provided text. Topics we like to focus on include: Main idea, supporting
      details, Point of view, Theme, Sequence, Elements of fiction (setting, characters,
      BME)'
title_generation:
  prompt: Write a title with fewer than 11 words to concisely describe this selection.
```

## Usage 
### Convert E-book to chunked CSV or TXT

#### 1. Use automated script to split your `pdf` or `epub`.
```bash
python3 book2text.py ebook-name.epub # or ebook-name.pdf (Epub is preferred)
```

**This step produces two outputs**:
a) `out/ebook-name.csv` (split by chapter or section)\
b) `out/ebook-name_processed.csv` (chunked)

***or***

#### 2. Place each chunk compressed to a single line of a text file, surrounded by double quotes.

### Generate Summary

`$``python3 sum.py --help`

```bash
Usage: python sum.py [OPTIONS] input_file

Options:
-c, --csv        Process a CSV file. Expected columns: Title, Text
-t, --txt        Process a text file. Each line should be a separate text chunk.
-m, --model      Model name to use for generation (default from config)
-p, --prompt     Alias of the prompt to use from config (default from config)
--help           Show this help message and exit.

For CSV input:
- Ensure your CSV has 'Title' and 'Text' columns.

For Text input:
- Each line should be a chunk of text surrounded by double quote.

The output CSV will include:
- Title: Final title chosen or generated
- Gen: Boolean indicating if the title was generated
- Text: Original input text
- model_name: Generated output
- Time: Processing time in seconds
- Len: Length of the output
```    

If you have your defaults set, then all you need is to specify which type of input, manual `text`, or automated `csv`. 
```
python3 sum.py -c ebook-name_processed.csv
```

***or*** 

In the following example, I've used `tools-prototype/split_pdf.py` to split the pdf not only by chapter but also subsection (producing `ebook-name_extracted.csv`), then manually process that output to place each chunk on a single line surrounded by double quote.

```
python3 sum.py -t ebook-name_extracted.csv
```

**This step generates two outputs**:
- `ebook-name_extracted_processed_sum.md` (rendered markdown)
- `ebook-name_extracted_processed_sum.csv` (csv with: input text, flattened md output, generation time, output length)


## Models
Download from one of two sources:

### Ollama
You can get any of them them right from ollama, template in all.
example: `ollama pull obook_summary:q5_k_m`

- [obook_summary](https://ollama.com/cognitivetech/obook_summary) - On Ollama.com
  - `latest` • 7.7GB • Q_8
  - `q3_k_m` • 3.5GB
  - `q4_k_m` • 4.4GB
  - `q5_k_m` • 5.1GB
  - `q6_k` • 5.9GB
- [obook_title](https://ollama.com/cognitivetech/obook_title) - On Ollama.com
  - `latest` • 7.7GB • Q_8
  - `q3_k_m` • 3.5GB
  - `q4_k_m` • 4.4GB
  - `q5_k_m` • 5.1GB
  - `q6_k`   • 5.9GB 

### HuggingFace
There is also complete weights, lora and ggguf on huggingface
- [Mistral Instruct Bulleted Notes](https://huggingface.co/collections/cognitivetech/mistral-instruct-bulleted-notes-v02-66b6e2c16196e24d674b1940) - Collection on HuggingFace
  - [cognitivetech/Mistral-7B-Inst-0.2-Bulleted-Notes](https://huggingface.co/cognitivetech/Mistral-7B-Inst-0.2-Bulleted-Notes)
  - [cognitivetech/Mistral-7b-Inst-0.2-Bulleted-Notes_GGUF](https://huggingface.co/cognitivetech/cognitivetech/Mistral-7b-Inst-0.2-Bulleted-Notes_GGUF)
  - [cognitivetech/Mistral-7B-Inst-0.2_Bulleted-Notes_LoRA](https://huggingface.co/cognitivetech/cognitivetech/Mistral-7B-Inst-0.2_Bulleted-Notes_LoRA)

## Check your eBook for clickable ToC.

Here you can see how to check whethere your eBook as the proper formatting, or not. **With ePub it should fail gracefully**.

\* In some rare occasion, even with clickable toc the script will not find that.

### Firefox
![image](https://github.com/user-attachments/assets/fc618e8c-d3e7-4bbd-aa16-1830fdc75b12)

### Brave 
![image](https://github.com/user-attachments/assets/c4491208-f66b-45cf-9095-f2f919d0fa49)

## Disclaimer

You are responsible for verifying that the summary tool creates an accurate summary. There are a variety of issues which can interfere with a quality summary, and if you aren't paying attention may slip your notice.

**1. References:**

Personally, I don't trust references from my fine-tuned model without verifying them manually. Maybe this is solved in newer models, but during my testing phase I noticed some bad references with 7b models I was using. I never tested this out to see the quality of the app on references, my personal preference is to remove any long references sections before summarizing, and deal with those separate. I don't think this is a permenant blocker, just an area that I haven't fully dealt with or understood, yet.

**2. Other:**

There are a few other things to watch out for. 

One of the reasons I keep the length of the input and output on CSV is that makes it easy to check when a summary is longer than the input, thats a red flag.

when the structure of the summary greatly deviates from the others, this can indicate issues with the summary. Some of these can be realated to special characters, or if the input is too long and the AI just doesn't grasp it all.


## Inspiration

The inspiration for this app was my intention to manually summarize a dozen books so I could tie together psychological theory and practice which they discuss and make a cohesive argument based on that information.

I've already read the books a few times, but now I need easy access to the information within so that I can relate it to others in a cohesive fashion.

Originally, after working at it this project manually, for a week, I was only a few chapters into my first book, I could see this was going to take a loong time.

Over the next 6 months I began learning how to use LLM, discovering were the best for my task, with fine-tuning to deliver production quality consistency in the results.

Now with this tool, I'm able to review a lot more material more quickly. This is a content curation tool that empowers me to not only learn things but more readily share that knowledge, without having to spend ages that it takes to create quality content.

Moreover, it can be used to create custom datasets based on whatever source materials you throw at it.

