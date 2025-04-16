# Movie Recommendation System Report

## 1. Introduction

Movie recommendation systems have become an essential component of modern entertainment platforms, helping users discover content relevant to their interests amid an overwhelming array of choices. These systems analyze user behavior and preferences to suggest movies that align with individual tastes, thereby enhancing user experience, increasing engagement, and driving content consumption.

In this project, we implemented and compared three distinct recommendation approaches:

- **User-Based Collaborative Filtering**: This approach identifies users with similar movie preferences and recommends movies highly rated by these similar users. It operates on the principle that users with similar past preferences will have similar future preferences.

- **Item-Based Collaborative Filtering**: This method focuses on the relationships between movies rather than users, recommending movies that are similar to those a user has already enjoyed. It's particularly effective in scenarios where the items (movies) change less frequently than user preferences.

- **Random Walk-Based Recommendation**: A graph-based approach inspired by Pinterest's Pixie algorithm, which represents users and movies as a bipartite graph and uses stochastic random walks to discover relevant recommendations through direct and indirect connections.

## 2. Dataset Description

### The MovieLens 100K Dataset

This project utilizes the MovieLens 100K dataset, a widely used benchmark collection in recommendation system research. The dataset contains:

- **943 unique users**
- **1,682 unique movies**
- **100,000 ratings** (1-5 scale)

The dataset is spread across three main files:
- `u.data`: Contains user-movie ratings with timestamp information
- `u.item`: Contains movie metadata including titles and release dates
- `u.user`: Contains user demographic information

### Key Features Used

For our recommendation models, we primarily used:
- User IDs: Unique identifiers for each user
- Movie IDs: Unique identifiers for each movie
- Ratings: Integer values from 1-5 representing user preferences
- Movie titles: For displaying meaningful recommendations

### Preprocessing Steps

Several preprocessing steps were performed to prepare the data:
1. Converting Unix timestamps to readable datetime format, revealing that the ratings were collected between September 1997 and April 1998
2. Checking for missing values across all datasets (minimal missing data was found)
3. Creating a user-movie matrix where rows represent users, columns represent movies, and cells contain ratings
4. Handling missing values in the matrix by:
   - Keeping missing values as NaN for initial exploration
   - Filling missing values with movie average ratings for the collaborative filtering approaches

## 3. Methodology

### User-Based Collaborative Filtering

User-based collaborative filtering operates by finding users with similar taste profiles and leveraging their preferences to generate recommendations:

1. **Similarity Calculation**: We used cosine similarity to measure how similar each pair of users is based on their rating patterns. This metric effectively captures preference alignment regardless of rating volume differences.

2. **Neighbor Selection**: For a target user, we identified the most similar users (excluding the target user themself) to serve as recommendation sources.

3. **Rating Prediction and Recommendation**: We calculated a weighted average of similar users' ratings for movies the target user hasn't watched, with greater weight given to more similar users. Movies were then ranked by these predicted ratings.

The key advantage of this approach is its ability to capture niche preferences shared among a small group of similar users, potentially uncovering non-obvious recommendations.

### Item-Based Collaborative Filtering

Item-based collaborative filtering shifts focus from user similarities to item similarities:

1. **Transposing the Matrix**: We transposed the user-movie matrix to make movies the rows and users the columns, enabling us to analyze movie-movie relationships.

2. **Item Similarity Computation**: Using cosine similarity, we measured the similarity between pairs of movies based on their rating patterns across users.

3. **Recommendation Generation**: For a given movie, we identified other movies with high similarity scores and recommended them as similar content.

This approach tends to be more stable than user-based filtering since movie characteristics typically change less frequently than user preferences, making it computationally efficient for systems with many more users than items.

### Random Walk-Based Pixie Algorithm

The Pixie-inspired approach represents the recommendation problem as a graph-based model:

1. **Graph Construction**: We created a bipartite graph where:
   - Users and movies are represented as nodes
   - An edge exists between a user and movie if the user rated that movie
   - The adjacency list representation efficiently stored these connections

2. **Random Walk Process**: Starting from a node (user or movie), the algorithm:
   - Randomly selects a connected node to visit next
   - Continues this process for a specified number of steps
   - Keeps track of how often each movie node is visited

3. **Recommendation Generation**: Movies are ranked based on their visitation frequency during the random walks, with more frequently visited movies considered more relevant.

This approach excels at discovering serendipitous recommendations by exploring both direct connections and more distant relationships in the graph structure.

## 4. Implementation Details

### User-Based Collaborative Filtering Implementation

The implementation followed these key steps:

1. Creating a user-movie matrix from the ratings data
2. Computing cosine similarity between all pairs of users
3. For a target user, finding the top similar users and extracting their ratings
4. For each movie not rated by the target user, calculating a weighted average rating based on similar users' ratings
5. Returning the top-ranked movies as recommendations

The function `recommend_movies_for_user(user_id, num=5)` encapsulates this logic, taking a user ID and desired number of recommendations as inputs and returning a DataFrame of recommended movies.

### Item-Based Collaborative Filtering Implementation

The item-based implementation follows a similar pattern with key differences:

1. Computing the item similarity matrix by applying cosine similarity to the transposed user-movie matrix
2. For a given movie, retrieving its similarity scores with all other movies
3. Sorting these similarities and selecting the top movies (excluding the input movie)
4. Mapping movie IDs to titles and returning a ranked list of recommendations

The function `recommend_movies(movie_name, num=5)` handles this process, taking a movie name and returning similar movies.

### Graph-Based Recommendation Implementation

The graph-based approach required more complex implementation:

1. **Adjacency List Creation**:
   ```python
   graph = {}
   for _, row in ratings.iterrows():
       user, movie = row['user_id'], row['movie_id']
       if user not in graph:
           graph[user] = set()
       if movie not in graph:
           graph[movie] = set()
       graph[user].add(movie)
       graph[movie].add(user)
   ```
   This created a dictionary where keys are nodes (users or movies) and values are sets of connected nodes.

2. **Random Walk Implementation**:
   - The `weighted_pixie_recommend` function performs multiple random walks starting from either a user or movie node
   - At each step, it randomly selects a neighboring node to visit
   - It maintains a counter for visited movie nodes
   - After completing all walks, it ranks movies by visit frequency

3. **Movie Ranking**:
   - Movies are ranked based on how frequently they were visited during the random walks
   - The top N movies are returned as recommendations
   - The algorithm can start from either a user node (for user-based recommendations) or a movie node (for "movies like this" recommendations)

## 5. Results and Evaluation

### Example Outputs

1. **User-Based Collaborative Filtering** for user 10:
   ```
   Ranking | Movie Name
   --------|----------------------------------
   1       | In the Company of Men (1997)
   2       | Misérables, Les (1995)
   3       | Thin Blue Line, The (1988)
   4       | Braindead (1992)
   5       | Boys, Les (1997)
   ```

2. **Item-Based Collaborative Filtering** for "Jurassic Park (1993)":
   ```
   Ranking | Movie Name
   --------|----------------------------------
   1       | Top Gun (1986)
   2       | Empire Strikes Back, The (1980)
   3       | Raiders of the Lost Ark (1981)
   4       | Indiana Jones and the Last Crusade (1989)
   5       | Speed (1994)
   ```

3. **Random Walk-Based Recommendations** for user 1:
   ```
   Ranking | Movie Name
   --------|----------------------------------
   1       | City of Lost Children, The (1995)
   2       | African Queen, The (1951)
   3       | Angels and Insects (1995)
   4       | English Patient, The (1996)
   5       | Lost Highway (1997)
   ```

### Comparative Analysis

1. **User-Based Collaborative Filtering**:
   - **Strengths**: Effectively captures personalized preferences; particularly good at identifying niche interests
   - **Weaknesses**: Performance degrades with sparse data; computationally intensive for large user bases
   - The recommendations for user 10 show an interesting pattern of art house and independent films, suggesting this user may have sophisticated taste in drama

2. **Item-Based Collaborative Filtering**:
   - **Strengths**: More stable than user-based; handles the "cold-start" problem better for new users
   - **Weaknesses**: Less personalized; tends to recommend popular items
   - The recommendations for "Jurassic Park" clearly capture the action-adventure blockbuster genre, showing strong content-based similarity

3. **Random Walk-Based Recommendations**:
   - **Strengths**: Discovers serendipitous recommendations; captures both direct and indirect relationships
   - **Weaknesses**: Requires parameter tuning; results can vary between runs due to randomness
   - The diverse recommendations for user 1 include both art films and classics, demonstrating the algorithm's ability to explore different areas of the taste space

### Limitations

1. **Data Sparsity**: The user-movie matrix is extremely sparse (approximately 6.3% density), which affects the quality of similarity calculations.

2. **Cold Start Problem**: All models struggle with new users or movies with few ratings.

3. **Scalability**: The current implementations, particularly the user-based approach, would face computational challenges with larger datasets.

4. **Evaluation Metrics**: The implementation lacks quantitative evaluation metrics (such as precision, recall, or RMSE) to objectively compare the different approaches.

5. **Parameter Sensitivity**: The random walk approach is sensitive to parameters like walk length and number of iterations, which were chosen somewhat arbitrarily.

## 6. Conclusion

### Key Takeaways

1. **Complementary Approaches**: Each recommendation method offers distinct strengths and captures different aspects of user preferences:
   - User-based filtering excels at personal recommendations based on similar users
   - Item-based filtering effectively captures content similarity
   - Graph-based approaches discover non-obvious connections through network exploration

2. **Graph Representation Power**: The bipartite graph representation provides a powerful framework for modeling complex user-item interactions, enabling the discovery of recommendations that might be missed by traditional matrix-based approaches.

3. **Algorithmic Trade-offs**: There's a clear trade-off between computational complexity, recommendation diversity, and accuracy across the different methods.

### Potential Improvements

1. **Hybrid Models**: Combining multiple recommendation approaches could leverage their complementary strengths:
   - Using item-based for new users and user-based for users with established profiles
   - Incorporating content-based features (genres, actors, directors) alongside collaborative filtering
   - Weighting different algorithms based on confidence scores

2. **Enhanced Graph Models**:
   - Incorporating edge weights based on rating values
   - Adding temporal decay to prioritize recent interactions
   - Including additional node types (genres, actors) for a heterogeneous graph

3. **Advanced Techniques**:
   - Implementing matrix factorization methods like SVD or NMF
   - Applying deep learning approaches such as neural collaborative filtering
   - Utilizing contextual information to provide situational recommendations

### Real-World Applications

1. **Streaming Platforms**: Netflix, Amazon Prime, and Disney+ could implement these algorithms to enhance content discovery and increase user engagement.

2. **E-commerce**: Online retailers can use similar approaches for product recommendations, particularly the graph-based methods which excel at discovering complementary items.

3. **Social Networks**: Friend recommendations and content suggestions on platforms like Facebook and Twitter benefit from graph-based approaches similar to our Pixie implementation.

4. **News Aggregators**: Personalized article recommendations can leverage user-based collaborative filtering to suggest content based on readers with similar interests.

5. **Music Streaming**: Services like Spotify can use item-based filtering to recommend songs similar to those in users' playlists.

In conclusion, this project demonstrates the effectiveness of different recommendation approaches and highlights how they can be implemented using the MovieLens dataset. The combination of collaborative filtering and graph-based methods provides a comprehensive solution to the recommendation problem, offering both personalized and diverse suggestions to enhance the user experience.