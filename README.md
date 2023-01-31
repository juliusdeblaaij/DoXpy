# DoXpy: An Objective Metric for Explainable AI

**NB This documentation will be updated soon**

DoXpy is a pip-installable python library giving you all that is necessary to objectively estimate the amount of explainability of any English piece of information (i.e. the output of an Explainable AI), scaling pretty well from single paragraphs to whole sets of documents.

DoXpy relies on deep language models for
- Answer Retrieval
- Sentence Embedding

This is why DoXpy supports several state-of-the-art deep learning libraries for NLP, making it easy to change deep language models at wish.
The supported libraries are:
- [Spacy](https://spacy.io)
- [Huggingface](https://huggingface.co)
- [TensorFlow Hub](https://www.tensorflow.org/hub/)
- [SentenceTransformers](https://www.sbert.net/index.html)

DoXpy is model-agnostic, which means that it can work with the output of any AI, if in English, regardless of its specific characteristics.
To do so, DoXpy exploits a specific theoretical model from Ordinary Language Philosophy called Achinstein's Theory of Explanations. 
You may find a thorough discussion of the underlying theory in this paper [An Objective Metric for Explainable AI: How and Why to Estimate the Degree of Explainability](http://arxiv.org/abs/2109.05327).
  
## Installation
This project has been tested on Debian 9 and macOS Mojave 10.14 with Python 3.7. 
The script [setup_virtualenv.sh](setup_virtualenv.sh) can install DoXpy and all its dependencies in a python3.7 virtualenv.

You can also install DoXpy by downloading this repo and running from within it: 
`pip install  --use-deprecated=legacy-resolver -e doxpy --no-cache-dir`

Before being able to run the [setup_virtualenv.sh](setup_virtualenv.sh) script, you have to install: virtualenv, python3-dev, python3-pip and make. 

For a simple example of how to use DoXpy, please consider the script [simple_example.py](simple_example.py).

## What to know
DoXpy is a python library allowing you to measure different aspects of explainability, such as fruitfulness, exactness and similarity to the explanandum.

As shown in [demo/assess_degree_of_explainability.py](demo/assess_degree_of_explainability.py), DoXpy relies on three main components to work:
- a [Knowledge Graph Extractor](doxpy/doxpy/models/knowledge_extraction/knowledge_graph_extractor.py)
- an [Answer Retriever](doxpy/doxpy/models/reasoning/answer_retriever.py)
- a [DoX Estimator](doxpy/doxpy/models/estimation/dox_estimator.py)

The Knowledge Graph Extractor can analyse any set of English documents (allowed formats: '.akn', '.html', '.pdf', '.json'), extracting from them a knowledge graph of information units.

The knowledge graph is then given as input to the Answer Retriever. 
Hence, the Answer Retriever is used by the DoX Estimator for computing the (average) DoX.

For the DoX Estimator to compute the DoX, it is necessary to define what is to be explained (i.e., the explanandum, the set of aspects to be explained). This may be done by selecting all the nodes in the previously extracted knowledge graph (as shown in [demo/assess_degree_of_explainability.py](demo/assess_degree_of_explainability.py)).

The DoX is a set of numbers. Each one of these numbers approximatively describes how well the analysed information can explain, on average, all the explanandum aspects by answering a pre-defined set of archetypal questions (e.g. WHY, HOW, WHEN, WHO) deemed to be sufficient to capture most of the explanation goals.

Considering that the DoX is a set, it is not sortable. 
This is why you need to average the values in the set as an average DoX. 

The default archetypes (or archetypal questions) used for the computation of DoX are the values of AnswerRetriever:: archetypal_questions_dict, and they are the following:
- What is a description of {X}?
- What is {X}?		
- What is {X}?
- Who {X}?
- Whom {X}?
- Why {X}?
- What if {X}?
- What is {X} for?
- How {X}?
- How much {X}?
- Where {X}?
- When {X}?
- Who by {X}?
- Which {X}?
- Whose {X}?
- In what manner {X}?
- What is the reason {X}?
- What is the result of {X}?
- What is an example of {X}?
- After what {X}?
- While what {X}?
- In what case {X}?
- Despite what {X}?
- What is contrasted with {X}?
- Before what {X}?
- Since when {X}?
- What is similar to {X}?
- Until when {X}?
- Instead of what {X}?
- What is an alternative to {X}?
- Except when {X}?
- Unless what {X}?

It is possible to manually specify a different list of archetypal questions, as shown in [demo/assess_degree_of_explainability.py](demo/assess_degree_of_explainability.py).

For more about why we selected all these archetypes and many more details, please read [An Objective Metric for Explainable AI: How and Why to Estimate the Degree of Explainability](http://arxiv.org/abs/2109.05327). 

## How to use DoXpy in your project
To use DoXpy on your project, you need to install it first. 
Then you can import it as in [demo/assess_degree_of_explainability.py](demo/assess_degree_of_explainability.py).
You may also need to configure the KnowledgeGraphExtractor,  the AnswerRetriever, and the DoXEstimator.

An example of import extracted from the script mentioned above is the following:
```
from doxpy.models.knowledge_extraction.knowledge_graph_extractor import KnowledgeGraphExtractor
from doxpy.models.estimation.dox_estimator import DoXEstimator
from doxpy.models.knowledge_extraction.knowledge_graph_manager import KnowledgeGraphManager
from doxpy.models.reasoning.answer_retriever import AnswerRetriever
```

An example of configuration options is the following:
```
ARCHETYPE_FITNESS_OPTIONS = {
	'only_overview_exploration': False,
	'answer_pertinence_threshold': 0.55, 
	'answer_to_question_max_similarity_threshold': 0.9502, 
	'answer_to_answer_max_similarity_threshold': 0.9502, 
}
OVERVIEW_OPTIONS = {
	'answer_pertinence_threshold': None, # default is None
	'answer_to_question_max_similarity_threshold': None, # default is 0.9502
	'answer_to_answer_max_similarity_threshold': None, # default is 0.9502
	'minimise': False,
	'sort_archetypes_by_relevance': False,
	'set_of_archetypes_to_consider': None, # set(['why','how'])
	'answer_horizon': None,
	'remove_duplicate_answer_sentences': True,

	'top_k': 100,
	'include_super_concepts_graph': False, 
	'include_sub_concepts_graph': True, 
	'add_external_definitions': False, 
	'consider_incoming_relations': True,
	'tfidf_importance': 0,
}

KG_MANAGER_OPTIONS = {
	# 'spacy_model': 'en_core_web_trf',
	# 'n_threads': 1,
	# 'use_cuda': True,
	'with_cache': False,
	'with_tqdm': False,
}

QA_EXTRACTOR_OPTIONS = {
	# 'models_dir': '/home/toor/Desktop/data/models', 
	'models_dir': '/Users/toor/Documents/University/PhD/Project/YAI/code/libraries/QuAnsX/data/models', 
	'use_cuda': True,

	'sbert_model': {
		'url': 'facebook-dpr-question_encoder-multiset-base', # model for paraphrase identification
		'cache_dir': '/Users/toor/Documents/Software/DLModels/sb_cache_dir',
		'use_cuda': True,
	},
}

KG_BUILDER_DEFAULT_OPTIONS = {
	'spacy_model': 'en_core_web_trf',
	'n_threads': 1,
	'use_cuda': True,

	'max_syntagma_length': None,
	'lemmatize_label': False,

	'default_similarity_threshold': 0.75,
	'tf_model': {
		'url': 'https://tfhub.dev/google/universal-sentence-encoder-large/5', # Transformer
		# 'url': 'https://tfhub.dev/google/universal-sentence-encoder/4', # DAN
		# 'cache_dir': '/public/francesco_sovrano/DoX/Scripts/.env',
		'cache_dir': '/Users/toor/Documents/Software/DLModels/tf_cache_dir',
		'use_cuda': False,
	},
	'with_centered_similarity': True,
}

CONCEPT_CLASSIFIER_DEFAULT_OPTIONS = {
	# 'spacy_model': 'en_core_web_trf',
	# 'n_threads': 1,
	# 'use_cuda': True,

	'tf_model': {
		'url': 'https://tfhub.dev/google/universal-sentence-encoder-large/5', # Transformer
		# 'url': 'https://tfhub.dev/google/universal-sentence-encoder/4', # DAN
		# 'cache_dir': '/public/francesco_sovrano/DoX/Scripts/.env',
		'cache_dir': '/Users/toor/Documents/Software/DLModels/tf_cache_dir',
		'use_cuda': False,
	},
	'with_centered_similarity': True,
	'default_similarity_threshold': 0.75,
	# 'default_tfidf_importance': 3/4,
}

SENTENCE_CLASSIFIER_DEFAULT_OPTIONS = {
	# 'spacy_model': 'en_core_web_trf',
	# 'n_threads': 1,
	# 'use_cuda': True,

	# 'tf_model': {
	# 	# 'url': 'https://tfhub.dev/google/universal-sentence-encoder-qa2/3', # English QA
	# 	'url': 'https://tfhub.dev/google/universal-sentence-encoder-multilingual-qa/3', # Multilingual QA # 16 languages (Arabic, Chinese-simplified, Chinese-traditional, English, French, German, Italian, Japanese, Korean, Dutch, Polish, Portuguese, Spanish, Thai, Turkish, Russian)
	# 	# 'url': 'https://tfhub.dev/google/LAReQA/mBERT_En_En/1',
	# 	'cache_dir': '/Users/toor/Documents/Software/DLModels/tf_cache_dir/',
	# 	'use_cuda': True,
	# }, 
	'sbert_model': {
		'url': 'facebook-dpr-question_encoder-multiset-base', # model for paraphrase identification
		# 'cache_dir': '/public/francesco_sovrano/DoX/Scripts/.env',
		'cache_dir': '/Users/toor/Documents/Software/DLModels/sb_cache_dir',
		'use_cuda': True,
	},
	'with_centered_similarity': False,
	'with_topic_scaling': False,
	'with_stemmed_tfidf': False,
	# 'default_tfidf_importance': 1/4,
}
```

## Experiments
Part of the experiments discussed in [An Objective Metric for Explainable AI: How and Why to Estimate the Degree of Explainability](http://arxiv.org/abs/2109.05327) can be run by executing the script [run_dox_assessment.sh](run_dox_assessment.sh).
This script runs the automatic assessment of the DoX for the two experiments on all the different versions of the two considered AI-based systems.
The results of the automatic assessment can be found at [demo/logs](demo/logs).

On the other hand, the code of the applications adopted for the user study is available at this GitHub repository: [YAI4Hu](https://github.com/Francesco-Sovrano/YAI4Hu).

The 2 XAI-based systems are: 
- a Heart Disease Predictor (HD)
- a Credit Approval System (CA)

For each of these systems, we have three different versions:
- a Normal AI-based Explanation (NAE): showing only the bare output of an AI together with some extra realistic contextual information, therefore making no use of any XAI.
- a Normal XAI-only Explainer (NXE): showing the output of an XAI and the information given by NAE.
- a 2nd-Level Exhaustive Explanatory Closure (2EC): the output of NXE connected to a 2nd (non-expandable) level of information consisting of an exhaustive and verbose set of autonomous static explanatory resources in the form of web pages.

The adopted AI are:
- [XGBoost](https://dl.acm.org/doi/abs/10.1145/2939672.2939785) for HD
- a Neural Network for CA

The adopted XAI are:
- [TreeSHAP](https://www.nature.com/articles/s42256-019-0138-9) for HD
- [CEM](https://dl.acm.org/doi/abs/10.5555/3326943.3326998) for CA

The two experiments are the following:
- The 1st compares the degree of explainability of NAE and NXE, expecting NXE to be the best because it relied on XAI.
- The 2nd compares the degree of explainability of NXE and 2EC, expecting 2EC to be the best because 2EC can explain many more things than NXE.

## Citations
This code is free. So, if you use this code anywhere, please cite us:
```
@article{sovrano2021metric,
  title={An Objective Metric for Explainable AI: How and Why to Estimate the Degree of Explainability},
  author={Sovrano, Francesco and Vitali, Fabio},
  journal={arXiv preprint arXiv:2109.05327},
  url={https://arxiv.org/abs/2109.05327},
  year={2021}
}
```

Thank you!

## Support
For any problem or question, please contact me at `cesco.sovrano@gmail.com`
