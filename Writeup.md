# Approach
The system has two stages of filter and classification:
    1. An embedding similarity for reducing the number of candidates that will be sent to the LLM. 
        -The companies profile were stored into a single text and then encoded with the "all-MiniLM-L6-v2" model
        -I used cosine similarity to produce a score for each query and the companies profile.
        -I also extracted some defining words that represents the query for a better LLM interpretation. 
        -After we received the score, from all of the candidates that were above the threshold(0.3), I took the first 20 in terms of score
    2. An LLM "judge" that returns the final results:
        -LLM used: Groq API with llama-3.1-8b-instant
        -The top candidates list and extracted information(defining words, profiles, etc.) were received in JSON format for a better interpretation
        - The model assings each company a score: 1.0 - match, 0.5 - partial match, 0 - no match
    In terms of efficiency and scalability, this system provides a very good option because it uses the strengths of embedding similarity(low cost and speed) and avoiding it's weaknesses(poor intent understanding) by sending the relevant candidates to a LLM model that can understand the context and judge accordingly.
# Tradeoffs
The system is optimized for:
    -speed: The llama-3.1-8b-instant is fast and low cost, but it lacks in terms of accuracy 
    -Cost: Groq is free and it forces the system to cap the candidates list and the description of companies. Because of its low price(free) we pay with the accuracy
    - Simplicity: The system is not a very complicated and complex one. It uses a basic embedding and cosine similarity algorithm instead of a more complex approach
    -Scalability: The system runs an O(N) complexity so it will work for a larger data set, but it might miss some relevant candidates.
# Error Analysis
The system has some limitation most of them because of its low cost, but they could be easily diminished. Examples:
    -If the query is a location + industry type, the location term dominates the similarity score and providing possibly not the best candidates.(Ex: Logistic companies in Romania)
    -The LLM is a free model and sometimes can misunderstanding some concepts or misclassify them due to its lower capacity
    -Special symbols, like diacritics for example, give a hard time to the model, but it was solved by normalizing the text
    -The description can't be above a certain number of chars and because of that we are losing precious context
# Scaling
For 100,000 companies some of the changes I would make are:
    -Pre-compute embeddings to reduce time and memory
    -Before the similarity test, I would implement a filter for the non-descriptive information(ex: revenue, address, number of employees, etc. )
    -Increase the number of candidates and the details provided
    -Upgrade the LLM for a better interpretation and more capacity for provided information
# 3.5 Failure Modes
In some cases, the system provides confident results but sometimes they might be incorrect, for example: 
    -If a word appears in some descriptive information of the company, it will be score as a match even if the actual primary business is not exactly the one that we are looking for
    -Queries that are too generic like "Companies before 2010" or "Companies in America" might produce some unrelated companies, all because the embedding score will be too high for certain candidates
    -Missing or incomplete data can be a challenge for the LLM to understand and judge correctly
    -Sometimes, duplicate companies are returned with different ID
To be monitored:
    -If the score is 1.0 for most companies, something is wrong. The LLM is being too generous and it provides greater scores than it should.
    -If the results indicates duplicates with different IDs, it indicates a data quality issue that was taking care of
    -If the results are NULL, the threshold is too high and must be slightly lowered
    -Groq API can sometimes produce rate limit error, but we dealt with it by putting a 30 seconds waiting time

