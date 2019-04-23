+++
title = "A Simple, Efficient, Parallelizable Algorithm for Approximated Nearest Neighbors."
date = 2018-05-21
authors = ["Sebastián Ferrada", "Benjamin Bustos", "Nora Reyes"]
publication_types = ["1"]
abstract = "The use of the join operator in metric spaces leads to what is known as a similarity join, where objects of two datasets are paired if they are  somehow  similar. We propose an heuristic that solves the 1-NN self-similarity join, that is, a similarity join of a dataset with itself, that brings together each element with its nearest neighbor within the same dataset. Solving the problem using a simple brute-force algorithm requires $O(n^2)$ distance calculations, since it requires to compare every element against all others. We propose a simple divide-and-conquer algorithm that gives an approximated solution for the self-similarity join that computes only $O(n^\\frac{3}{2}) $ distances. We show how the algorithm can be easily modified in order to improve the precision up to 31% (i.e., the percentage of correctly found 1-NNs) and such that 79% of the results are within the 10-NN, with no significant extra distance computations. We present how the algorithm can be executed in parallel and prove that using $\\Theta(\\sqrt{n})$ processors, the total execution takes linear time. We end discussing ways in which the algorithm can be improved in the future."
featured = false
publication = "*AMW*"
url_pdf = "pdf/simjoinAMW18.pdf"
url_code = "https://github.com/scferrada/self-sim-join"
+++

