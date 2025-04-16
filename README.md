# Movie Recommendation System

A comprehensive implementation of multiple recommendation algorithms using the MovieLens 100K dataset.

## Project Overview

This project implements and compares three different recommendation approaches:
- **User-Based Collaborative Filtering**: Recommends movies based on similar users' preferences
- **Item-Based Collaborative Filtering**: Recommends movies similar to those a user has enjoyed
- **Pixie-Inspired Random Walk Algorithm**: Uses graph-based methods to discover recommendations through network exploration

## Dataset

The MovieLens 100K dataset contains:
- 943 unique users
- 1,682 unique movies
- 100,000 ratings (1-5 scale)

## Repository Contents

- `Programming_Assignment_1.ipynb`: Jupyter notebook with all implementations
- `Recommendation_Report.md`: Detailed report on methodology and findings
- Data files:
  - `ratings.csv`, `movies.csv`, `users.csv`: Processed data files

## Setup and Installation

```bash
# Clone the repository
git clone https://github.com/gcnabingc/movie_recommendation_sys.git

# Install required packages
pip install pandas numpy scikit-learn
```

## Usage Examples

```python
# User-based recommendation
recommendations = recommend_movies_for_user(10, num=5)

# Item-based recommendation
recommendations = recommend_movies("Jurassic Park (1993)", num=5)

# Random walk-based recommendation
recommendations = weighted_pixie_recommend(1, walk_length=15, num=5)
```

## Key Findings

- **User-Based Filtering**: Excels at finding niche interests shared among similar users
- **Item-Based Filtering**: Provides more stable recommendations that capture genre preferences
- **Graph-Based Approach**: Discovers serendipitous recommendations by exploring indirect connections

Each method offers complementary strengths, suggesting that hybrid approaches could leverage the best aspects of all three techniques.
