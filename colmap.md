変更点は以下の通りです：
メインの変更： sequential_matcher から exhaustive_matcher に変更しました。

sequential_matcher：隣接する画像同士のマッチングのみを行うため、高速ですが連続したシーケンスに最適化されています
exhaustive_matcher：すべての画像ペアの組み合わせでマッチングを行うため、より強力な再構成が得られます

処理時間の考慮： exhaustive_matcherはより多くのペア比較を行うため、処理時間が長くなります。特に画像枚数が多い場合は注意が必要です。画像枚数が数百枚を超える場合は、別のマッチング戦略を検討する必要があるかもしれません。


```

def run_colmap_reconstruction(image_dir, colmap_dir):
    """Estimate camera poses and 3D point cloud with COLMAP"""
    print("Running SfM reconstruction with COLMAP...")

    database_path = os.path.join(colmap_dir, "database.db")
    sparse_dir = os.path.join(colmap_dir, "sparse")
    os.makedirs(sparse_dir, exist_ok=True)

    # Set environment variable
    env = os.environ.copy()
    env['QT_QPA_PLATFORM'] = 'offscreen'

    # Feature extraction
    print("1/4: Extracting features...")
    subprocess.run([
        'colmap', 'feature_extractor',
        '--database_path', database_path,
        '--image_path', image_dir,
        '--ImageReader.single_camera', '1',
        '--ImageReader.camera_model', 'OPENCV',
        '--SiftExtraction.use_gpu', '0'  # Use CPU
    ], check=True, env=env)

    # Feature matching
    print("2/4: Matching features...")
    subprocess.run([
        'colmap', 'exhaustive_matcher',#'sequential_matcher',  # Use sequential_matcher instead of exhaustive_matcher
        '--database_path', database_path,
        '--SiftMatching.use_gpu', '0'  # Use CPU
    ], check=True, env=env)

    # Sparse reconstruction
    print("3/4: Sparse reconstruction...")
    subprocess.run([
        'colmap', 'mapper',
        '--database_path', database_path,
        '--image_path', image_dir,
        '--output_path', sparse_dir,
        '--Mapper.ba_global_max_num_iterations', '20',  # Speed up
        '--Mapper.ba_local_max_num_iterations', '10'
    ], check=True, env=env)

    # Export to text format
    print("4/4: Exporting to text format...")
    model_dir = os.path.join(sparse_dir, '0')
    if not os.path.exists(model_dir):
        # Use the first model found
        subdirs = [d for d in os.listdir(sparse_dir) if os.path.isdir(os.path.join(sparse_dir, d))]
        if subdirs:
            model_dir = os.path.join(sparse_dir, subdirs[0])
        else:
            raise FileNotFoundError("COLMAP reconstruction failed")

    subprocess.run([
        'colmap', 'model_converter',
        '--input_path', model_dir,
        '--output_path', model_dir,
        '--output_type', 'TXT'
    ], check=True, env=env)

    print(f"COLMAP reconstruction complete: {model_dir}")
    return model_dir

def convert_cameras_to_pinhole(input_file, output_file):
    """Convert camera model to PINHOLE format"""
    print(f"Reading camera file: {input_file}")

    with open(input_file, 'r') as f:
        lines = f.readlines()

    converted_count = 0
    with open(output_file, 'w') as f:
        for line in lines:
            if line.startswith('#') or line.strip() == '':
                f.write(line)
            else:
                parts = line.strip().split()
                if len(parts) >= 4:
                    cam_id = parts[0]
                    model = parts[1]
                    width = parts[2]
                    height = parts[3]
                    params = parts[4:]

                    # Convert to PINHOLE format
                    if model == "PINHOLE":
                        f.write(line)
                    elif model == "OPENCV":
                        # OPENCV: fx, fy, cx, cy, k1, k2, p1, p2
                        fx = params[0]
                        fy = params[1]
                        cx = params[2]
                        cy = params[3]
                        f.write(f"{cam_id} PINHOLE {width} {height} {fx} {fy} {cx} {cy}\n")
                        converted_count += 1
                    else:
                        # Convert other models too
                        fx = fy = max(float(width), float(height))
                        cx = float(width) / 2
                        cy = float(height) / 2
                        f.write(f"{cam_id} PINHOLE {width} {height} {fx} {fy} {cx} {cy}\n")
                        converted_count += 1
                else:
                    f.write(line)

    print(f"Converted {converted_count} cameras to PINHOLE format")

def prepare_gaussian_splatting_data(image_dir, colmap_model_dir):
    """Prepare data for Gaussian Splatting"""
    print("Preparing data for Gaussian Splatting...")

    data_dir = f"{WORK_DIR}/data/video"
    os.makedirs(f"{data_dir}/sparse/0", exist_ok=True)
    os.makedirs(f"{data_dir}/images", exist_ok=True)

    # Copy images
    print("Copying images...")
    img_count = 0
    for img_file in os.listdir(image_dir):
        if img_file.lower().endswith(('.jpg', '.jpeg', '.png')):
            shutil.copy(
                os.path.join(image_dir, img_file),
                f"{data_dir}/images/{img_file}"
            )
            img_count += 1
    print(f"Copied {img_count} images")

    # Convert and copy camera file to PINHOLE format
    print("Converting camera model to PINHOLE format...")
    convert_cameras_to_pinhole(
        os.path.join(colmap_model_dir, 'cameras.txt'),
        f"{data_dir}/sparse/0/cameras.txt"
    )

    # Copy other files
    for filename in ['images.txt', 'points3D.txt']:
        src = os.path.join(colmap_model_dir, filename)
        dst = f"{data_dir}/sparse/0/{filename}"
        if os.path.exists(src):
            shutil.copy(src, dst)
            print(f"Copied {filename}")
        else:
            print(f"Warning: {filename} not found")

    print(f"Data preparation complete: {data_dir}")
    return data_dir

def train_gaussian_splatting(data_dir, iterations=7000):
    """Train the Gaussian Splatting model"""
    print(f"Training Gaussian Splatting model for {iterations} iterations...")

    model_path = f"{WORK_DIR}/output/video"

    cmd = [
        sys.executable, 'train.py',
        '-s', data_dir,
        '-m', model_path,
        '--iterations', str(iterations),
        '--eval'
    ]

    subprocess.run(cmd, cwd=WORK_DIR, check=True)

    return model_path

def render_video(model_path, output_video_path, iteration=3000):
    """Generate video from the trained model"""
    print("Rendering video...")

    # Execute rendering
    cmd = [
        sys.executable, 'render.py',
        '-m', model_path,
        '--iteration', str(iteration)
    ]

    subprocess.run(cmd, cwd=WORK_DIR, check=True)

    # Find the rendering directory
    possible_dirs = [
        f"{model_path}/test/ours_{iteration}/renders",
        f"{model_path}/train/ours_{iteration}/renders",
    ]

    render_dir = None
    for test_dir in possible_dirs:
        if os.path.exists(test_dir):
            render_dir = test_dir
            print(f"Rendering directory found: {render_dir}")
            break

    if render_dir and os.path.exists(render_dir):
        render_imgs = sorted([f for f in os.listdir(render_dir) if f.endswith('.png')])

        if render_imgs:
            print(f"Found {len(render_imgs)} rendered images")

            # Create video with ffmpeg
            subprocess.run([
                'ffmpeg', '-y',
                '-framerate', '30',
                '-pattern_type', 'glob',
                '-i', f"{render_dir}/*.png",
                '-c:v', 'libx264',
                '-pix_fmt', 'yuv420p',
                '-crf', '18',
                output_video_path
            ], check=True)

            print(f"Video saved: {output_video_path}")
            return True

    print("Error: Rendering directory not found")
    return False

def create_gif(video_path, gif_path):
    """Create GIF from MP4"""
    print("Creating animated GIF...")

    subprocess.run([
        'ffmpeg', '-y',
        '-i', video_path,
        '-vf', 'setpts=8*PTS,fps=10,scale=720:-1:flags=lanczos',
        '-loop', '0',
        gif_path
    ], check=True)

    if os.path.exists(gif_path):
        size_mb = os.path.getsize(gif_path) / (1024 * 1024)
        print(f"GIF creation complete: {gif_path} ({size_mb:.2f} MB)")
        return True

    return False

```
