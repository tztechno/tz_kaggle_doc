このコードから**汎用性の高い関数**をいくつか抽出できます：

## 1. **汎用BFS距離計算関数**
```python
def bfs_distance(grid, start, goal, max_dist=None):
    """任意のグリッド上の2点間の最短距離を計算"""
    if start == goal:
        return 0
    
    # マンハッタン距離で早期枝刈り
    manhattan = abs(start[0] - goal[0]) + abs(start[1] - goal[1])
    if max_dist and manhattan > max_dist:
        return max_dist
    
    queue = deque([(start[0], start[1], 0)])
    visited = set([start])
    directions = [(-1,0), (1,0), (0,-1), (0,1)]
    
    while queue:
        x, y, dist = queue.popleft()
        
        if max_dist and dist >= max_dist:
            return max_dist
            
        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            if (0 <= nx < len(grid) and 0 <= ny < len(grid[0]) and 
                (nx, ny) not in visited and grid[nx][ny] == '.'):  # 通行可能条件をパラメータ化可能
                if (nx, ny) == goal:
                    return dist + 1
                visited.add((nx, ny))
                queue.append((nx, ny, dist + 1))
    
    return max_dist if max_dist else float('inf')  # 到達不能
```

**汎用性**：
- 任意のグリッドサイズに対応
- 障害物条件をパラメータ化可能
- 最大距離制限でパフォーマンス調整

## 2. **到達可能性チェック関数**
```python
def reachability_check(grid, start, goal, temporary_blocks=None, max_steps=100):
    """一時的な障害物を考慮した到達可能性チェック"""
    if start == goal:
        return True
    
    if temporary_blocks is None:
        temporary_blocks = []
    
    # 一時的にブロック
    originals = []
    for pos in temporary_blocks:
        if 0 <= pos[0] < len(grid) and 0 <= pos[1] < len(grid[0]):
            originals.append((pos, grid[pos[0]][pos[1]]))
            grid[pos[0]][pos[1]] = 'X'  # ブロック記号
    
    try:
        queue = deque([start])
        visited = set([start])
        directions = [(-1,0), (1,0), (0,-1), (0,1)]
        
        while queue and max_steps > 0:
            max_steps -= 1
            x, y = queue.popleft()
            
            for dx, dy in directions:
                nx, ny = x + dx, y + dy
                if (0 <= nx < len(grid) and 0 <= ny < len(grid[0]) and 
                    (nx, ny) not in visited and grid[nx][ny] != 'X'):  # ブロック以外は通行可能
                    
                    if (nx, ny) == goal:
                        return True
                    
                    visited.add((nx, ny))
                    queue.append((nx, ny))
        
        return False
    
    finally:
        # 必ず復元
        for pos, original in originals:
            grid[pos[0]][pos[1]] = original
```

**汎用性**：
- 一時的なグリッド変更を安全に処理
- 例外安全な設計（finally節）
- 様々なゲームや経路探索に適用可能

## 3. **ビームサーチフレームワーク**
```python
class BeamState:
    """汎用ビームサーチ状態クラス"""
    def __init__(self, solution, score, data=None):
        self.solution = solution  # 現在の解
        self.score = score        # 評価スコア
        self.data = data          # 追加データ
        
    def __lt__(self, other):
        return self.score > other.score  # 降順ソート用

def beam_search(initial_state, generate_successors, evaluate_state, 
                beam_width=5, max_depth=10):
    """汎用ビームサーチアルゴリズム"""
    beam = [initial_state]
    best_state = initial_state
    
    for depth in range(max_depth):
        next_beam = []
        
        for state in beam:
            for successor in generate_successors(state):
                score = evaluate_state(successor)
                new_state = BeamState(
                    successor.solution, 
                    score, 
                    successor.data
                )
                
                next_beam.append(new_state)
                
                if score > best_state.score:
                    best_state = new_state
        
        if not next_beam:
            break
            
        next_beam.sort()
        beam = next_beam[:beam_width]
    
    return best_state.solution
```

**汎用性**：
- 任意の問題に適用可能なフレームワーク
- 状態生成関数と評価関数を外部から注入
- 組み合わせ最適化問題全般に使用可能

## 4. **候補点選択の汎用関数**
```python
def select_candidate_positions(grid, start, goal, condition_fn, score_fn, 
                             search_radius=5, max_candidates=20):
    """条件とスコア関数に基づいて候補位置を選択"""
    candidates = []
    N = len(grid)
    
    start_x = max(0, min(start[0], goal[0]) - search_radius)
    end_x = min(N, max(start[0], goal[0]) + search_radius + 1)
    start_y = max(0, min(start[1], goal[1]) - search_radius)
    end_y = min(N, max(start[1], goal[1]) + search_radius + 1)
    
    for i in range(start_x, end_x):
        for j in range(start_y, end_y):
            if condition_fn(grid, i, j, start, goal):
                score = score_fn(grid, i, j, start, goal)
                if score > 0:
                    candidates.append((score, i, j))
    
    candidates.sort(reverse=True)
    return candidates[:max_candidates]
```

## 再利用可能な要素

1. **グリッドベースの経路探索**：様々なグリッドゲームに適用可能
2. **ビームサーチフレームワーク**：組合せ最適化問題全般に使用可能
3. **安全なグリッド操作**：一時的な変更と自動復元
4. **候補生成パターン**：類似の問題での再利用が容易

これらの関数は、**経路探索ゲーム、ロボットナビゲーション、組合せ最適化問題**など、幅広い分野で再利用できる汎用性を持っています。
