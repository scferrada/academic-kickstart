+++
title = "Similarity Joins in SPARQL"
date = 2020-02-06T12:00:00  # Schedule page publish date.
draft = false

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
time_start = 2020-02-06T12:00:00
#time_end = 2020-02-06T13:00:00

# Abstract and optional shortened version.
abstract = "RDF datasets are often made accessible over the Web through a SPARQL endpoint where users typically write queries requesting matches on the content. For instance, in Wikidata, a SPARQL query may request 'the names and nationalities of laureates of the Nobel Prize in Literature that have fought in a war'. However, there are times when an exact match is not what users need; instead they wish to write a similarity query, for which there is no declarative support in the SPARQL standard. Hence a query to obtain 'the Latin American country with the most similar population and GDP to Italy cannot be written declaratively in SPARQL; even if the query were expressed with distances defined using low-level numeric operators, the SPARQL engine is unlikely to know how to optimise such a query, resorting to brute-force evaluation. On the other hand, the potential applications for similarity queries in SPARQL are numerous, including: entity comparison and linking, multimedia retrieval, similarity graphs, pattern recognition, data integration, as well as domain use-cases, such as protein similarity computation. In this talk, we motivate similarity-based queries for SPARQL and present a system for executing such queries over RDF graphs and integrating them with SPARQL including syntax discussion, query planning and optimisation, and novel use-cases."
abstract_short = ""

# Name of event and optional event URL.
event = "INSIGHT Meetings"
event_url = ""

# Location of event.
location = "Lower Dangan, Newcastle, Galway, Ireland"

# Is this a selected talk? (true/false)
selected = false

# Projects (optional).
#   Associate this talk with one or more of your projects.
#   Simply enter the filename (excluding '.md') of your project file in `content/project/`.
#   E.g. `projects = ["deep-learning"]` references `content/project/deep-learning.md`.
projects = [""]

# Tags (optional).
#   Set `tags = []` for no tags, or use the form `tags = ["A Tag", "Another Tag"]` for one or more tags.
tags = ["Similarity Joins", "SPARQL", "Web Image Retrieval", "Semantic Web"]

# Links (optional).
url_pdf = ""
url_slides = "/pptx/insight-02-2020.pptx"
url_video = ""
url_code = ""

# Does the content use math formatting?
math = false

# Does the content use source code highlighting?
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++
