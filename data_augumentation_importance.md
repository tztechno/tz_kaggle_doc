```
import numpy as np
import pandas as pd
from sklearn.ensemble import RandomForestRegressor
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
import warnings
warnings.filterwarnings('ignore')

def get_feature_importance(X, y, method='rf'):
    """特徴量重要度を計算"""
    if method == 'rf':
        model = RandomForestRegressor(n_estimators=100, random_state=42)
        model.fit(X, y)
        return model.feature_importances_
    elif method == 'linear':
        scaler = StandardScaler()
        X_scaled = scaler.fit_transform(X)
        model = LinearRegression()
        model.fit(X_scaled, y)
        return np.abs(model.coef_) / np.sum(np.abs(model.coef_))

def correlation_aware_noise(X_df, features, n_new=50, noise_level=0.01, random_state=42):
    """相関を考慮したノイズ追加"""
    np.random.seed(random_state)
    X_values = X_df[features].values
    y_values = X_df['BeatsPerMinute'].values
    
    # 特徴量間の相関行列を計算
    corr_matrix = np.corrcoef(X_values, rowvar=False)
    
    # コレスキー分解で相関を保持するノイズを生成
    try:
        L = np.linalg.cholesky(corr_matrix + np.eye(len(features)) * 1e-6)
    except:
        # 正定値でない場合は固有値分解を使用
        eigenvals, eigenvecs = np.linalg.eigh(corr_matrix)
        eigenvals = np.maximum(eigenvals, 1e-6)
        L = eigenvecs @ np.diag(np.sqrt(eigenvals)) @ eigenvecs.T
    
    X_new = []
    y_new = []
    
    for _ in range(n_new):
        idx = np.random.randint(0, len(X_values))
        base_sample = X_values[idx]
        
        # 相関を保持するノイズを生成
        uncorr_noise = np.random.normal(0, noise_level, size=len(features))
        corr_noise = L @ uncorr_noise
        
        # 元のスケールに合わせてノイズを調整
        feature_stds = np.std(X_values, axis=0)
        scaled_noise = corr_noise * feature_stds
        
        new_sample = base_sample + scaled_noise
        
        # 範囲内にクリップ
        for i, feature in enumerate(features):
            new_sample[i] = np.clip(new_sample[i], 
                                   X_values[:, i].min(), 
                                   X_values[:, i].max())
        
        # yの値も相関に基づいて調整
        y_corr = np.corrcoef(X_values.T, y_values)[:-1, -1]
        y_change = np.sum(scaled_noise * y_corr) * 0.1  # 調整係数
        new_y = y_values[idx] + y_change
        
        X_new.append(new_sample)
        y_new.append(new_y)
    
    X_new_df = pd.DataFrame(X_new, columns=features)
    X_new_df['BeatsPerMinute'] = y_new
    return X_new_df

def pca_based_augmentation(X_df, features, n_new=50, noise_level=0.01, random_state=42):
    """PCA空間でのデータ増幅"""
    np.random.seed(random_state)
    X_values = X_df[features].values
    y_values = X_df['BeatsPerMinute'].values
    
    # PCAで主成分を計算
    scaler = StandardScaler()
    X_scaled = scaler.fit_transform(X_values)
    pca = PCA()
    X_pca = pca.fit_transform(X_scaled)
    
    # 主成分の重要度に基づいてノイズレベルを調整
    explained_var_ratio = pca.explained_variance_ratio_
    
    X_new = []
    y_new = []
    
    for _ in range(n_new):
        idx = np.random.randint(0, len(X_pca))
        base_pca = X_pca[idx].copy()
        
        # 各主成分の重要度に応じてノイズを追加
        for i in range(len(base_pca)):
            component_noise_level = noise_level * np.sqrt(explained_var_ratio[i])
            base_pca[i] += np.random.normal(0, component_noise_level)
        
        # 元の特徴量空間に戻す
        X_scaled_new = pca.inverse_transform(base_pca)
        X_new_sample = scaler.inverse_transform(X_scaled_new.reshape(1, -1))[0]
        
        # 範囲内にクリップ
        for i in range(len(features)):
            X_new_sample[i] = np.clip(X_new_sample[i],
                                     X_values[:, i].min(),
                                     X_values[:, i].max())
        
        # yの値を元のサンプルベースで調整
        y_new_sample = y_values[idx] + np.random.normal(0, noise_level * np.std(y_values))
        
        X_new.append(X_new_sample)
        y_new.append(y_new_sample)
    
    X_new_df = pd.DataFrame(X_new, columns=features)
    X_new_df['BeatsPerMinute'] = y_new
    return X_new_df

def importance_weighted_noise(X_df, features, n_new=50, noise_level=0.01, random_state=42):
    """特徴量重要度に基づいて重み付けしたノイズ追加"""
    np.random.seed(random_state)
    X_values = X_df[features].values
    y_values = X_df['BeatsPerMinute'].values
    
    # 特徴量重要度を計算
    importance = get_feature_importance(X_values, y_values, method='rf')
    
    X_new = []
    y_new = []
    
    for _ in range(n_new):
        idx = np.random.randint(0, len(X_values))
        base_sample = X_values[idx].copy()
        
        # 重要度が低い特徴量により多くのノイズを追加
        # 重要度が高い特徴量には少ないノイズを追加
        for i in range(len(features)):
            # 重要度が高いほどノイズレベルを小さく
            adjusted_noise_level = noise_level * (1 - importance[i] * 0.8)
            noise = np.random.normal(0, adjusted_noise_level * np.std(X_values[:, i]))
            base_sample[i] += noise
            
            # 範囲内にクリップ
            base_sample[i] = np.clip(base_sample[i],
                                   X_values[:, i].min(),
                                   X_values[:, i].max())
        
        # yの値も調整（重要な特徴量の変化により敏感に）
        y_change = 0
        for i in range(len(features)):
            feature_change = base_sample[i] - X_values[idx, i]
            y_change += feature_change * importance[i] * 0.1
        
        y_new_sample = y_values[idx] + y_change + np.random.normal(0, noise_level * np.std(y_values) * 0.5)
        
        X_new.append(base_sample)
        y_new.append(y_new_sample)
    
    X_new_df = pd.DataFrame(X_new, columns=features)
    X_new_df['BeatsPerMinute'] = y_new
    return X_new_df

def gaussian_mixture_augmentation(X_df, features, n_new=50, noise_level=0.01, random_state=42):
    """ガウシアン混合モデルベースのデータ増幅"""
    from sklearn.mixture import GaussianMixture
    
    np.random.seed(random_state)
    X_values = X_df[features].values
    y_values = X_df['BeatsPerMinute'].values
    
    # 全体データ（X + y）でガウシアン混合モデルを学習
    full_data = np.column_stack([X_values, y_values])
    
    # 最適なコンポーネント数を決定（簡単のため固定値を使用）
    n_components = min(5, len(X_values) // 10)
    gmm = GaussianMixture(n_components=n_components, random_state=random_state)
    gmm.fit(full_data)
    
    # 新しいサンプルを生成
    new_samples, _ = gmm.sample(n_new)
    
    X_new = new_samples[:, :-1]
    y_new = new_samples[:, -1]
    
    # 範囲内にクリップ
    for i in range(len(features)):
        X_new[:, i] = np.clip(X_new[:, i],
                             X_values[:, i].min(),
                             X_values[:, i].max())
    
    y_new = np.clip(y_new, y_values.min(), y_values.max())
    
    X_new_df = pd.DataFrame(X_new, columns=features)
    X_new_df['BeatsPerMinute'] = y_new
    return X_new_df

def evaluate_feature_preservation(original_X, original_y, augmented_X, augmented_y, features):
    """特徴量重要度の保持度を評価"""
    orig_importance = get_feature_importance(original_X, original_y)
    aug_importance = get_feature_importance(augmented_X, augmented_y)
    
    # 重要度の相関係数を計算
    correlation = np.corrcoef(orig_importance, aug_importance)[0, 1]
    
    # 重要度の差の平均絶対誤差
    mae = np.mean(np.abs(orig_importance - aug_importance))
    
    print(f"特徴量重要度の相関: {correlation:.3f}")
    print(f"重要度の平均絶対誤差: {mae:.3f}")
    print("\n特徴量別重要度比較:")
    for i, feature in enumerate(features):
        print(f"{feature}: {orig_importance[i]:.3f} -> {aug_importance[i]:.3f}")
    
    return correlation, mae

# 使用例
if __name__ == "__main__":
    # サンプルデータの作成（実際のデータに置き換えてください）
    np.random.seed(42)
    n_samples = 100
    features = ['danceability', 'energy', 'acousticness', 'valence']
    
    # サンプルデータ生成
    X_sample = np.random.rand(n_samples, len(features))
    y_sample = (X_sample[:, 0] * 0.5 + X_sample[:, 1] * 0.3 + 
                X_sample[:, 2] * 0.1 + X_sample[:, 3] * 0.1 + 
                np.random.normal(0, 0.1, n_samples)) * 100 + 100
    
    X_df_sample = pd.DataFrame(X_sample, columns=features)
    X_df_sample['BeatsPerMinute'] = y_sample
    
    print("=== 各手法の特徴量重要度保持性能 ===\n")
    
    methods = [
        ("相関考慮ノイズ", correlation_aware_noise),
        ("PCAベース", pca_based_augmentation),
        ("重要度重み付け", importance_weighted_noise),
        ("ガウシアン混合", gaussian_mixture_augmentation)
    ]
    
    for method_name, method_func in methods:
        print(f"--- {method_name} ---")
        augmented = method_func(X_df_sample, features, n_new=50)
        
        original_X = X_df_sample[features].values
        original_y = X_df_sample['BeatsPerMinute'].values
        aug_X = augmented[features].values
        aug_y = augmented['BeatsPerMinute'].values
        
        evaluate_feature_preservation(original_X, original_y, aug_X, aug_y, features)
        print()

```        
