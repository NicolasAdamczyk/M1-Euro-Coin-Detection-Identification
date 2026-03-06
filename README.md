# Euro Coin Detection and Identification

**Master 1 Computer Science — IFLBX030 Introduction to Image Analysis**

## Objective

Detect and identify euro coins in an image in order to:
- Count the number of coins
- Estimate the total monetary value

The project relies on classical image processing techniques with minimal machine learning (KNN), without deep learning.

## Project Structure

```
coin_detection.py          # Detection: coin localization
coin_identification.py     # Identification: classification and denomination
coin_detector.py           # Orchestrator: full pipeline
main.py                    # Main script (execution + evaluation)
ground_truth.csv           # Ground truth (106 images)
requirements.txt           # Python dependencies
data/                      # Images to analyze
results/                   # Generated results

# Supporting notebooks explaining design choices
notebook_Detection.ipynb        # Detailed segmentation and detection steps
notebook_Identification.ipynb   # Colorimetric analysis (LAB/HSV)
notebook_Evaluation.ipynb       # In-depth statistical analysis of results
```

## Usage

```bash
pip install -r requirements.txt

# Process all images + evaluation
python main.py

# Process a single image
python main.py data/exemple1.jpg
```

## Pipeline

### 1. Preprocessing
- Redimensionnement (largeur cible = 800px, hauteur max = 1000px)
- Amelioration du contraste via CLAHE
- Flou gaussien

### 2. Analyse du fond
- Echantillonnage des pixels de bordure en HSV
- Classification du fond : clair/sombre et colore/neutre

### 3. Segmentation
- **Fond neutre** : seuillage d'Otsu sur l'image en niveaux de gris + operations morphologiques
- **Fond colore** : seuillage d'Otsu sur le canal de saturation (les pieces metalliques ont une saturation bien plus faible que les surfaces colorees)

### 4. Detection
Trois methodes combinees :
- **Hough Circle Transform** : detection de cercles avec balayage multi-parametres
- **Watershed** : separation des pieces qui se touchent via la transformee de distance
- **Contours** : detection par analyse de contours pour les pieces bien separees

Les resultats sont fusionnes (strategie "le plus de pieces gagne"), puis filtres par NMS (suppression des doublons) et coherence des rayons.

### 5. Classification des couleurs
- Extraction de features : moyenne HSV, moyenne Lab (a\*, b\*), ecart-type de saturation, score bimetallique ameliore
- Classification par regles adaptatives selon le type de fond
- Raffinement optionnel par KNN (k=5, 30 exemples d'entrainement)
- Trois groupes : **cuivre** (1c, 2c, 5c), **or** (10c, 20c, 50c), **bimetallique** (1 euro, 2 euro)

### 6. Denomination
Attribution basee sur la taille relative au sein de chaque groupe de couleur, en utilisant les diametres reels des pieces en euro comme reference.

### 1. Preprocessing
- Resizing (target width = 800px, max height = 1000px)
- Contrast enhancement using CLAHE
- Gaussian blur

### 2. Background Analysis
- Sampling border pixels in HSV
- Background classification: light/dark and colored/neutral

### 3. Segmentation
- Neutral background: Otsu thresholding on the grayscale image + morphological operations
- Colored background: Otsu thresholding on the saturation channel (metal coins have much lower saturation than colored surfaces)

### 4. Detection
Three combined methods:
- Hough Circle Transform: circle detection with multi-parameter scanning
- Watershed: separation of touching coins using distance transform
- Contours: contour analysis for well-separated coins

Results are merged using a "most coins wins" strategy, then filtered with NMS (Non-Maximum Suppression) and radius consistency checks.

### 5. Color Classification
- Feature extraction: Mean HSV, Mean Lab (a*, b*), Saturation standard deviation, Improved bimetallic score
- Classification steps: Rule-based classification adapted to background type
- Optional refinement with KNN (k = 5, 30 training samples)
Three groups: Copper (1c, 2c, 5c), Gold (10c, 20c, 50c), Bimetallic (1 euro, 2 euro)

### 6. Denomination
Coins are assigned denominations based on relative size within each color group, using the real euro coin diameters as reference.

## Results

**Detection (106 images)**

| Metric         | Value  |
| -------------- | ------ |
| Precision      | 97.48% |
| Recall         | 84.01% |
| F1-Score       | 90.25% |
| Real coins     | 1107   |
| Detected coins | 954    |
| Exact counting | 56.60% |

**Identification / Financial Evaluation**

| Metric                     | Value              |
| -------------------------- | ------------------ |
| Financial MAE              | 1.90 EUR per image |
| Overall financial accuracy | 6.60%              |
| Average time per image     | 0.24s              |

**Per Group**

| Group | Images | MAE (EUR) | Exact Count |
| ----- | ------ | --------- | ----------- |
| gp1   | 14     | 0.62      | 64.29%      |
| gp2   | 15     | 0.92      | 40.00%      |
| gp3   | 10     | 5.59      | 60.00%      |
| gp4   | 10     | 0.65      | 70.00%      |
| gp5   | 25     | 2.99      | 32.00%      |
| gp6   | 10     | 1.57      | 90.00%      |
| gp7   | 12     | 0.58      | 83.33%      |
| gp8   | 10     | 1.89      | 50.00%      |

## Challenges Encountered

- **Fonds colores (rouge, vert)** : la segmentation classique en niveaux de gris echouait completement. Solution : basculer sur le canal de saturation HSV, ou le contraste metal/surface coloree est net.

- **Pieces qui se touchent** : la detection de Hough seule ne suffisait pas. L'ajout de l'algorithme Watershed a permis de separer les groupes de pieces.

- **Faux positifs / faux negatifs** : equilibrer les parametres de Hough (strict a relaxe) et la validation par contraste a necessite de nombreuses iterations. Un jeu de 5 parametres a ete conserve pour couvrir un maximum de cas.

- **Performance** : les premieres versions prenaient plus de 60s par image. L'optimisation principale a ete de pre-calculer les conversions d'image (gris, HSV) une seule fois et de les reutiliser dans la validation, ramenant le temps moyen a ~0.24s.

- **Regressions** : chaque amelioration pour un type d'image risquait de degrader les resultats sur d'autres. Une evaluation systematique sur les 106 images apres chaque modification a ete indispensable.

- **Colored backgrounds (red, green)**: classical grayscale segmentation failed completely.
- **Solution**: switch to the HSV saturation channel, where the metal vs colored surface contrast is much clearer.

- **Touching coins**: Hough detection alone was insufficient.
- **Solution**: Adding the Watershed algorithm enabled separation of clustered coins.

- **False positives / false negatives**: balancing Hough parameters (strict to relaxed) and contrast validation required many iterations.
- **Solution**: A set of 5 parameter configurations was retained to cover the widest range of cases.

- **Performance**: early versions took more than 60s per image.
- **Solution**: The main optimization was precomputing image conversions (grayscale, HSV) once and reusing them across validation steps, reducing average runtime to ~0.24s.

- **Regressions**: each improvement for one type of image risked degrading results on others.
- **Solution**:A systematic evaluation on all 106 images after every modification was essential.

## Dependencies

- OpenCV (`opencv-python`)
- NumPy
- scikit-learn (KNN only)
- Matplotlib (evaluation plots)
- Seaborn
- Pandas
