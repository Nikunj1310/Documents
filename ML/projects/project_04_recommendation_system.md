# Project 4 — Recommendation System

**Covers:** Chapters 3–8, Matrix Factorization (linear algebra from Ch 1)  
**Estimated time:** 1–2 weeks  
**Difficulty:** Intermediate

---

## Project Goal

Build a movie recommendation system from scratch, progressing from simple baselines to matrix factorization to neural collaborative filtering:

1. **Baseline:** Popularity-based & user-mean recommendations
2. **Collaborative Filtering:** Matrix factorization (SVD-based)
3. **Neural CF:** Embedding-based deep learning
4. **Evaluation:** Proper offline metrics for recommendations

---

## Dataset: MovieLens

```python
# MovieLens 100K: 100,000 ratings from 943 users on 1,682 movies
# MovieLens 1M: 1M ratings (use this if you have compute)

import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split

# Download: https://grouplens.org/datasets/movielens/100k/
ratings = pd.read_csv('ml-100k/u.data', sep='\t',
                       names=['user_id', 'item_id', 'rating', 'timestamp'])

movies = pd.read_csv('ml-100k/u.item', sep='|', encoding='latin-1',
                     names=['item_id', 'title', 'release_date', 'video_date',
                            'imdb_url'] + [f'genre_{i}' for i in range(19)],
                     usecols=['item_id', 'title'])

print(f"Users: {ratings['user_id'].nunique()}")
print(f"Items: {ratings['item_id'].nunique()}")
print(f"Ratings: {len(ratings)}")
print(f"Density: {len(ratings) / (ratings['user_id'].nunique() * ratings['item_id'].nunique()):.4f}")
print(ratings['rating'].describe())
```

### Train/Test Split

For recommendations, never split randomly — use temporal split or leave-last-out.

```python
# Leave-last-out: for each user, hold out their most recent rating
ratings = ratings.sort_values(['user_id', 'timestamp'])

def train_test_split_last(df):
    test = df.groupby('user_id').last().reset_index()
    train = df.merge(test[['user_id', 'item_id']], on=['user_id', 'item_id'], 
                     how='left', indicator=True)
    train = train[train['_merge'] == 'left_only'].drop('_merge', axis=1)
    return train, test

train, test = train_test_split_last(ratings)
print(f"Train: {len(train)}, Test: {len(test)}")
```

---

## Evaluation Metrics

```python
def precision_at_k(recommended, relevant, k=10):
    """Fraction of top-k recommendations that are relevant."""
    return len(set(recommended[:k]) & set(relevant)) / k

def recall_at_k(recommended, relevant, k=10):
    """Fraction of relevant items that appear in top-k."""
    if not relevant:
        return 0.0
    return len(set(recommended[:k]) & set(relevant)) / len(relevant)

def ndcg_at_k(recommended, relevant, k=10):
    """Normalized Discounted Cumulative Gain — rewards relevant items at top."""
    dcg = sum(1 / np.log2(i + 2) for i, item in enumerate(recommended[:k]) 
              if item in relevant)
    idcg = sum(1 / np.log2(i + 2) for i in range(min(k, len(relevant))))
    return dcg / idcg if idcg > 0 else 0.0

def evaluate_recommender(recommender, test_df, train_df, k=10):
    """Evaluate a recommender on precision, recall, NDCG@k."""
    results = {'precision': [], 'recall': [], 'ndcg': []}
    
    all_items = set(train_df['item_id'].unique())
    
    for user_id in test_df['user_id'].unique():
        relevant = set(test_df[test_df['user_id'] == user_id]['item_id'].values)
        already_seen = set(train_df[train_df['user_id'] == user_id]['item_id'].values)
        candidates = list(all_items - already_seen)
        
        recommendations = recommender.recommend(user_id, candidates, k=k)
        
        results['precision'].append(precision_at_k(recommendations, relevant, k))
        results['recall'].append(recall_at_k(recommendations, relevant, k))
        results['ndcg'].append(ndcg_at_k(recommendations, relevant, k))
    
    return {k: np.mean(v) for k, v in results.items()}
```

---

## Approach 1: Baselines

```python
class PopularityRecommender:
    """Recommend the most popular items (not seen by user)."""
    
    def fit(self, train_df):
        self.popular_items = (
            train_df.groupby('item_id')['rating'].count()
            .sort_values(ascending=False)
            .index.tolist()
        )
        return self
    
    def recommend(self, user_id, candidates, k=10):
        return [item for item in self.popular_items if item in set(candidates)][:k]

class UserMeanRecommender:
    """Recommend items with highest mean rating among users with similar average ratings."""
    
    def fit(self, train_df):
        self.item_mean = train_df.groupby('item_id')['rating'].mean()
        return self
    
    def recommend(self, user_id, candidates, k=10):
        scores = {item: self.item_mean.get(item, 0) for item in candidates}
        return sorted(scores, key=scores.get, reverse=True)[:k]

# Evaluate
pop = PopularityRecommender().fit(train)
print("Popularity:", evaluate_recommender(pop, test, train, k=10))
```

---

## Approach 2: Matrix Factorization

The core idea: factor the user-item rating matrix R into two lower-rank matrices:
```
R ≈ U × Vᵀ
```
- U ∈ ℝ^(|users| × d): user latent factors
- V ∈ ℝ^(|items| × d): item latent factors
- d: latent dimension (embedding size)

Predicted rating: `R̂_{u,i} = u_u · v_i + bias_u + bias_i + global_mean`

### Gradient Descent Matrix Factorization

```python
class MatrixFactorization:
    def __init__(self, n_factors=50, lr=0.01, reg=0.01, n_epochs=50, verbose=True):
        self.n_factors = n_factors
        self.lr = lr
        self.reg = reg
        self.n_epochs = n_epochs
        self.verbose = verbose
    
    def fit(self, train_df, val_df=None):
        self.user_ids = {u: i for i, u in enumerate(train_df['user_id'].unique())}
        self.item_ids = {it: i for i, it in enumerate(train_df['item_id'].unique())}
        
        n_users = len(self.user_ids)
        n_items = len(self.item_ids)
        
        # Initialize embeddings
        scale = 0.01
        self.U = np.random.randn(n_users, self.n_factors) * scale
        self.V = np.random.randn(n_items, self.n_factors) * scale
        self.bu = np.zeros(n_users)   # user biases
        self.bi = np.zeros(n_items)   # item biases
        self.global_mean = train_df['rating'].mean()
        
        # Prepare training data
        users = train_df['user_id'].map(self.user_ids).values
        items = train_df['item_id'].map(self.item_ids).values
        ratings = train_df['rating'].values.astype(float)
        
        for epoch in range(self.n_epochs):
            # SGD: shuffle and iterate
            idx = np.random.permutation(len(ratings))
            
            for i in idx:
                u, it, r = users[i], items[i], ratings[i]
                
                # Prediction
                r_hat = (self.global_mean + self.bu[u] + self.bi[it] +
                         self.U[u] @ self.V[it])
                error = r - r_hat
                
                # Update with L2 regularization
                self.bu[u] += self.lr * (error - self.reg * self.bu[u])
                self.bi[it] += self.lr * (error - self.reg * self.bi[it])
                self.U[u] += self.lr * (error * self.V[it] - self.reg * self.U[u])
                self.V[it] += self.lr * (error * self.U[u] - self.reg * self.V[it])
            
            if self.verbose and (epoch + 1) % 10 == 0:
                train_rmse = self._compute_rmse(train_df)
                msg = f"Epoch {epoch+1}: Train RMSE = {train_rmse:.4f}"
                if val_df is not None:
                    msg += f", Val RMSE = {self._compute_rmse(val_df):.4f}"
                print(msg)
        
        return self
    
    def predict(self, user_id, item_id):
        if user_id not in self.user_ids or item_id not in self.item_ids:
            return self.global_mean
        u = self.user_ids[user_id]
        it = self.item_ids[item_id]
        return self.global_mean + self.bu[u] + self.bi[it] + self.U[u] @ self.V[it]
    
    def recommend(self, user_id, candidates, k=10):
        scores = {item: self.predict(user_id, item) for item in candidates}
        return sorted(scores, key=scores.get, reverse=True)[:k]
    
    def _compute_rmse(self, df):
        preds = [self.predict(r['user_id'], r['item_id']) 
                 for _, r in df.iterrows()]
        return np.sqrt(np.mean((df['rating'].values - preds) ** 2))

mf = MatrixFactorization(n_factors=50, lr=0.005, reg=0.02, n_epochs=50)
mf.fit(train, test)
print("Matrix Factorization:", evaluate_recommender(mf, test, train, k=10))
```

---

## Approach 3: Neural Collaborative Filtering

```python
import torch
import torch.nn as nn
from torch.utils.data import Dataset, DataLoader

class RatingsDataset(Dataset):
    def __init__(self, df, user2idx, item2idx):
        self.users = df['user_id'].map(user2idx).values
        self.items = df['item_id'].map(item2idx).values
        self.ratings = df['rating'].values.astype(np.float32)
    
    def __len__(self): return len(self.ratings)
    
    def __getitem__(self, idx):
        return (torch.tensor(self.users[idx], dtype=torch.long),
                torch.tensor(self.items[idx], dtype=torch.long),
                torch.tensor(self.ratings[idx]))

class NeuralCF(nn.Module):
    def __init__(self, n_users, n_items, embed_dim=64, layers=[128, 64, 32]):
        super().__init__()
        
        # Embedding layers
        self.user_emb = nn.Embedding(n_users, embed_dim)
        self.item_emb = nn.Embedding(n_items, embed_dim)
        
        # MLP layers
        mlp_layers = []
        in_dim = embed_dim * 2
        for out_dim in layers:
            mlp_layers.extend([nn.Linear(in_dim, out_dim), nn.ReLU(), nn.Dropout(0.2)])
            in_dim = out_dim
        mlp_layers.append(nn.Linear(in_dim, 1))
        self.mlp = nn.Sequential(*mlp_layers)
        
        # Bias terms
        self.user_bias = nn.Embedding(n_users, 1)
        self.item_bias = nn.Embedding(n_items, 1)
        self.global_bias = nn.Parameter(torch.tensor(3.5))
        
        self._init_weights()
    
    def _init_weights(self):
        nn.init.normal_(self.user_emb.weight, std=0.01)
        nn.init.normal_(self.item_emb.weight, std=0.01)
        nn.init.zeros_(self.user_bias.weight)
        nn.init.zeros_(self.item_bias.weight)
    
    def forward(self, user_ids, item_ids):
        u = self.user_emb(user_ids)
        v = self.item_emb(item_ids)
        
        mlp_out = self.mlp(torch.cat([u, v], dim=-1))
        
        bias = (self.user_bias(user_ids) + self.item_bias(item_ids) + self.global_bias)
        
        return (mlp_out + bias).squeeze(-1)

# Training
all_users = list(sorted(ratings['user_id'].unique()))
all_items = list(sorted(ratings['item_id'].unique()))
user2idx = {u: i for i, u in enumerate(all_users)}
item2idx = {it: i for i, it in enumerate(all_items)}

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
ncf = NeuralCF(len(all_users), len(all_items)).to(device)

train_dataset = RatingsDataset(train, user2idx, item2idx)
train_loader = DataLoader(train_dataset, batch_size=512, shuffle=True, num_workers=4)

optimizer = torch.optim.Adam(ncf.parameters(), lr=1e-3, weight_decay=1e-5)
scheduler = torch.optim.lr_scheduler.StepLR(optimizer, step_size=10, gamma=0.5)
criterion = nn.MSELoss()

for epoch in range(30):
    ncf.train()
    total_loss = 0
    for users, items, ratings_batch in train_loader:
        users, items, ratings_batch = users.to(device), items.to(device), ratings_batch.to(device)
        
        preds = ncf(users, items)
        loss = criterion(preds, ratings_batch)
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        total_loss += loss.item()
    
    scheduler.step()
    if (epoch + 1) % 5 == 0:
        print(f"Epoch {epoch+1}: Loss = {total_loss/len(train_loader):.4f}")
```

---

## Approach 4: Item Embeddings for Exploration

After training the matrix factorization model, explore the learned item embeddings:

```python
# Visualize movie embeddings in 2D
from sklearn.manifold import TSNE

item_vectors = mf.V   # (n_items, n_factors)

tsne = TSNE(n_components=2, perplexity=30, random_state=42)
item_2d = tsne.fit_transform(item_vectors)

# Color by genre
genre_cols = [col for col in movies.columns if col.startswith('genre_')]
genre_names = ['unknown', 'Action', 'Adventure', 'Animation', "Children's",
               'Comedy', 'Crime', 'Documentary', 'Drama', 'Fantasy',
               'Film-Noir', 'Horror', 'Musical', 'Mystery', 'Romance',
               'Sci-Fi', 'Thriller', 'War', 'Western']

# Map item_ids to genres
# ... (join with movies dataframe)

plt.figure(figsize=(12, 10))
for i, genre in enumerate(genre_names):
    mask = (movie_genres[:, i] == 1)
    if mask.sum() > 10:
        plt.scatter(item_2d[mask, 0], item_2d[mask, 1], 
                    label=genre, alpha=0.6, s=10)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
plt.title('Movie Embeddings (t-SNE) — Colored by Genre')
plt.tight_layout()
plt.show()

# Find similar movies
def find_similar_movies(movie_id, top_k=10):
    movie_vec = mf.V[mf.item_ids[movie_id]]
    sims = mf.V @ movie_vec / (np.linalg.norm(mf.V, axis=1) * np.linalg.norm(movie_vec) + 1e-8)
    top_ids = sims.argsort()[-top_k-1:-1][::-1]
    return [list(mf.item_ids.keys())[i] for i in top_ids]

similar = find_similar_movies(50)  # "Star Wars"
print(movies[movies['item_id'].isin(similar)]['title'].tolist())
```

---

## Deliverables

- [ ] Baseline RMSE and ranking metrics documented
- [ ] Matrix Factorization achieving RMSE < 0.92 and NDCG@10 > 0.05
- [ ] Neural CF trained and evaluated
- [ ] Model comparison table (all approaches)
- [ ] Movie embedding visualization (t-SNE) — do movies cluster by genre?
- [ ] Top-10 recommendations for 5 different user profiles — do they seem reasonable?
- [ ] "Cold start" analysis: how does performance degrade for users with < 5 ratings?

## Questions to Answer

1. How does increasing the embedding dimension affect RMSE? Is there a point of diminishing returns?
2. What is the "cold start" problem? How would you handle a new user who has never rated anything?
3. Why do we evaluate with NDCG@10 rather than just RMSE?
4. What's the difference between collaborative filtering and content-based filtering?
5. How would you add side information (movie genres, user demographics) to improve recommendations?
