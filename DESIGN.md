# Overview
This project is a speculative design tool that generates climate-based future scenarios using real text data from IPCC reports. The tool reads and processes large volumes of paragraph-level text from scientific documents, tags the data with thematic and sentiment metadata, categorizes them by future probability (possible, plausible, probable), and then synthesizes original narratives based on user-selected filters. It is implemented entirely in Python and includes a modular backend pipeline, a narrative generation engine, and integration with a prebuilt React frontend.

## High-Level Architecture
The solution is structured into the following key stages:

1. **Data Ingestion**: Extract raw text from PDF sources using `PyMuPDF` and convert each page into a structured JSON format.
2. **Segmentation**: Break each page into paragraph-level units and assign unique IDs and capture metadata.
3. **Truncation**: Limit paragraph length to a fixed word count to ensure compatibility with embedding and the simple NLP model chosen for this prototype.
4. **Thematic Clustering**: Use BERTopic and `sentence-transformers` to assign each paragraph to a theme based on semantic similarity.
5. **Tagging**: Use KeyBERT for keyword extraction and a zero-shot classifier for sentiment tagging.
6. **Probability Classification**: Assign each theme a future type based on percentile frequency (probable, plausible, possible).
7. **Search Indexing**: Compile all tagged data into a `search_index.json` for fast retrieval and filtering.
8. **Narrative Generation**: Generate 3–5 sentence narratives using structured templates and real signals based on filters the user selected: positive/negative sentiment, and probable/possible/plausible futures.


## Design Decisions

### File Structure
The project uses `data/` and `scripts/` directory structure to separate assets from logic. I kept intermediate files (structured, truncated, tagged) for debugging and transparency. The final search index is written to a single JSON file for lightweight access by the frontend. 

### Use of BERTopic
I chose BERTopic for thematic clustering because it provides interpretable topics based on semantic embeddings, which matched the project’s need to assign general themes without manual labeling.

### Probability Model Based on Frequency
The distinction between “probable,” “plausible,” and “possible” futures was based on percentile thresholds of theme occurrence. This simple yet effective statistical approach gave the user interpretable insight into the commonality of signals across documents, without relying on subjective judgments.

### Signal Tagging with KeyBERT + Zero-Shot
KeyBERT was used to extract subthemes (keywords) using a max-sum similarity approach and a custom vectorizer to ensure consistent tokenization. Sentiment was determined using HuggingFace’s `facebook/bart-large-mnli` model via zero-shot classification, which avoided the need for fine-tuning or training on labeled data.

### Narrative Generation Strategy
Rather than build a generative model from scratch, I developed a prompt-style logic that uses a narrative template structure to build short stories. These narratives integrate signals naturally into fluid prose, inspired by speculative fiction writing styles. Each narrative ends with a stylized reflection sentence, chosen at random from a curated set.

### Indexing Approach
For simplicity, I chose to consolidate all filtered signals into a single `search_index.json`. This supports narrative generation and simple frontend integration. I would have preferred a API approach, which I aim to do in the next iteration. 

## Challenges and Tradeoffs
- **Issues with Narrative Generation**: The .py scripts should result in a somewhat coherent narrative output, but that doesn't always happen. At times the output is nonsensical. More time and a higher level of competency and familiarity with programming should help me identify the problem in a future iteration. I refrained from using AI to solve the problem for me and am opting to submit a solution that doesn't always output the desired content.The narrative generation issue impacts the front-end experience too.
- **Tagging Time**: The `tag_all_paragraphs.py` script was slow due to heavy NLP model use. I later added fallbacks and batching to mitigate this. But, in a future iteration I would want to explore a better performing data storage, extraction and tagging mechanism altogether to handle larger data volumes.
- **Memory Usage**: BERTopic and embedding models were memory-intensive. Truncating long paragraphs reduced risk of crashes and improved performance.
- **Narrative Fluidity**: Creating compelling speculative narratives required iteration. Early versions read like bullet point summaries. I refined the logic to weave in emotional tone, connectors, and variance in sentence structure.

## Future Improvements

- Fix the narrative generator issue - the output should always follow some logic and coherence.
- Swap out the static JSON index with an API-based search for scalability.
- Introduce a lightweight GUI for editing themes and signal classifications manually.
- Extend narrative logic to include geographic and actor-type filters from the metadata file.

## Final Thoughts
This project reflects a blend of creative storytelling and technical experimentation. I deliberately chose tools that challenged me in every stage of the prototyping process. The result is a working prototype of a speculative design engine that transforms real-world climate signals into engaging future narratives—quickly, flexibly, and with interpretability at its core.