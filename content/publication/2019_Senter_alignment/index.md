---
abstract: "Gene annotation has traditionally required direct comparison of DNA sequences between an unknown gene and a database of known ones using string comparison methods. However, these methods do not provide useful information when a gene does not have a close match in the database. In addition, each comparison can be costly when the database is large since it requires alignments and a series of string comparisons. In this work we propose a novel approach: using recurrent neural networks to embed DNA or amino-acid sequences in a low-dimensional space in which distances correlate with functional similarity. This embedding space overcomes both shortcomings of the method of aligning sequences and comparing homology. First, it allows us to obtain information about genes which do not have exact matches by measuring their similarity to other ones in the database. If our database is labeled this can provide labels for a query gene as is done in traditional methods. However, even if the database is unlabeled it allows us to find clusters and infer some characteristics of the gene population. In addition, each comparison is much faster than traditional methods since the distance metric is reduced to the Euclidean distance, and thus efficient approximate nearest neighbor algorithms can be used to find the best match. We present results showing the advantage of our algorithm. More specifically we show how our embedding can be useful for both classification tasks when our labels are known, and clustering tasks where our sequences belong to classes which have not been seen before."
authors:
- troyalty
- admin
date: "2019-09-17T00:00:00Z"
doi: ""
featured: true
image:
  caption: 
  focal_point: ""
  preview_only: false
projects: []
publication: 'arXiv'
publication_short: ""
publication_types:
- "3" 
publishDate: "2019-09-17T00:00:00Z"
slides: 
summary: 
tags:
- Source Themes
title: "Unaligned Sequence Similarity Search Using Deep Learning"
url_code: ""
url_dataset: ""
url_pdf: pdf/2019_Senter.pdf
url_poster: ""
url_project: ""
url_slides: ""
url_source: "https://arxiv.org/abs/1909.06929"
url_video: ""
---

