# Systeme-intelligent-ChestX-ray14-NIH-
État de l’art et data sets
Les datasets: /n
ChestX-ray14 (NIH) /n
Un ensemble de données comprenant 112,120 radiographies thoraciques provenant de 30,805
patients uniques. Chaque image est annotée avec jusqu'à 14 pathologies différentes incluant
pneumonie, atélectasie, cardiomégalie, épanchement pleural, infiltrat, masse, nodule,
pneumothorax, consolidation, oedème, emphysème, fibrose, épaississement pleural et hernie.
Les images sont accompagnées de métadonnées cliniques précieuses.
Lien : https://nihcc.app.box.com/v/ChestXray-NIHCC
Planning
<img width="1355" height="823" alt="image" src="https://github.com/user-attachments/assets/d321148c-14e4-48b9-af4e-a2351f9f298e" />
<img width="1347" height="291" alt="image" src="https://github.com/user-attachments/assets/bd07c105-44ef-430c-afa8-4afc27240411" />


# ÉTAPE 1 : Configuration Initiale et Importation
1.1 Importation des Bibliothèques

## Bibliothèques principales
- TensorFlow/Keras : Deep learning
- Pandas/NumPy : Manipulation données
- OpenCV/PIL : Traitement d'images
- Matplotlib/Seaborn : Visualisation
1.2 Configuration de Reproductibilité
Seeds fixes (42) pour tous les générateurs aléatoires

1.3 Paramètres Globaux
IMG_SIZE = (224, 224) : Standard pour ViT/ResNet

BATCH_SIZE = 32 : Équilibre mémoire/performance


# ÉTAPE 2 : Analyse des Métadonnées
2.1 Chargement des Données
df = pd.read_csv(CSV_PATH)  # 112,120 rows × 12 columns
2.2 Structure des Données
Colonnes principales analysées :

Image Index : Identifiant unique

Finding Labels : Diagnostics (multi-labels)

Patient ID, Patient Age, Patient Gender

View Position : PA/AP

2.3 Statistiques Démographiques
python
## Résultats clés :
n_patients = df["Patient ID"].nunique()  # 30,805 patients
n_images = len(df)                       # 112,120 images
ratio = n_images/n_patients              # 3.64 images/patient

## Distribution genre :
 M : 63,340 (56.5%)
 F : 48,780 (43.5%)
2.4 Analyse des Labels
python
df['n_labels'] = df['Finding Labels'].str.split('|').apply(len)
label_stats = df['n_labels'].describe()

## Résultats :
 Mean : 1.26 labels/image
 Max : 9 labels simultanés
 Single label : 91,324 images (81.5%)
 Multiple labels : 20,796 images (18.5%)

# ÉTAPE 3 : Nettoyage des Données
## 3.1 Validation des Colonnes Essentielles
python
expected_cols = {
    "Image Index", "Finding Labels", "Patient ID", 
    "Patient Age", "Patient Gender", "View Position"
}
missing = expected_cols - set(df.columns)  # Aucune colonne manquante
## 3.2 Traitement de l'Âge
python
 Conversion et validation
df["Patient Age"] = pd.to_numeric(df["Patient Age"], errors="coerce")

 Détection outliers
age_outliers = ((df["Patient Age"] < 0) | (df["Patient Age"] > 100)).sum()  # 16 outliers

 Remplacement des valeurs aberrantes
df.loc[(df["Patient Age"] < 0) | (df["Patient Age"] > 100), "Patient Age"] = np.nan
3.3 Normalisation du Genre
python
df["Patient Gender"] = df["Patient Gender"].astype(str).str.upper().str.strip()
invalid_gender = ~df["Patient Gender"].isin(["M","F"])  # 0 entrées invalides

## 3.4 Bilan du Nettoyage
Âge : 16 valeurs manquantes sur 112,120 (0.014%)

Genre : 100% de données valides

Structure finale : 112,120 × 13 colonnes

# ÉTAPE 4 : Analyse Exploratoire (EDA)
## 4.1 Préparation des Données pour Visualisation
python
Explosion des labels multiples
df["label_list"] = df["Finding Labels"].fillna("No Finding").str.split("|")
df_exploded = df.explode("label_list")
## 4.2 Visualisations Créées
## 4.2.1 Distribution des Labels par Image
Pattern : La majorité des images ont 1 label
Pic : 91,324 images avec un seul diagnostic
Queue : Décroissance exponentielle jusqu'à 9 labels

## 4.2.2 Distribution par Genre
Hommes : 56.5% des images
Femmes : 43.5% des images

Équilibre : Relativement équilibré

## 4.2.3 Distribution d'Âge
Plage : 1 à 100 ans (après nettoyage)
Pic : 45-55 ans (population adulte)
Shape : Distribution quasi-normale

## 4.2.4 Top 15 Pathologies
python
top_pathologies = df_exploded["label_list"].value_counts().head(15)
Pathologies dominantes :

No Finding
Infiltration
Effusion
Atelectasis
Nodule

## 4.2.5 View Position
PA : Majoritaire (standard clinique)
AP : Minoritaire (situations spécifiques)

## 4.2.6 Images par Patient
Distribution : Asymétrique à droite
Majorité : 1-5 images par patient
Outliers : Patients avec nombreuses suivis

## 4.3 Métriques Clés Calculées
python
print(f"Most common pathology: {top_pathologies.index[0]} ({top_pathologies.iloc[0]} cases)")
print(f"Images with 'No Finding': {(df_exploded['label_list'] == 'No Finding').sum()}")
print(f"Total unique pathologies: {df_exploded['label_list'].nunique()}")

# ÉTAPE 5 : Préparation pour le Deep Learning
## 5.1 Configuration Technique
Taille d'image : 224×224 pixels
Format : Compatible ViT et CNN standards
Batch processing : Optimisé pour GPU

# 5.2 Pipeline de Données Prévu
Chargement : Par patient (éviter les fuites)
Prétraitement : Normalisation, redimensionnement
Augmentation : Rotation, flip, ajustement contraste
Batching : Groupement efficace

# ANALYSE DES DEFIS TECHNIQUES IDENTIFIES
Défi 1 : Déséquilibre des Classes
Problème : Certaines pathologies très rares
Solution envisagée : Weighted loss functions, oversampling ciblé

Défi 2 : Multi-label Classification
Complexité : 18.5% des images avec diagnostics multiples
Approche : Sigmoid activation avec seuillage indépendant

Défi 3 : Fuite de Données
Risque : Images du même patient dans train/test
Solution : Split par Patient ID

Défi 4 : Qualité d'Image Variable
Observation : Résolutions et contrastes variables
Approche : Prétraitement standardisé

# INSIGHTS CLINIQUES EXTRAPOLES
1. Prévalence des Pathologies
Les pathologies pulmonaires obstructives et infectieuses dominent, reflétant la charge mondiale des maladies respiratoires.

2. Complexité Diagnostique
La forte proportion de diagnostics multiples suggère des comorbidités fréquentes en pneumologie.

3. Démographie des Patients
La distribution d'âge correspond aux populations à risque pour les pathologies pulmonaires.

RECOMMANDATIONS POUR LA SUITE
1. Stratégie de Modélisation
python
# Approche recommandée :
1. Model Baseline (ResNet50)
2. Vision Transformer
3. Model Ensemble
4. Explainable AI (Grad-CAM)
2. Validation Rigoureuse
Split : Par patient (80/10/10)

Métriques : AUC-ROC par classe, F1-score
Comparaison : Benchmark avec radiologues

3. Optimisations Techniques
Mixed precision : Acceleration GPU

Data loader : Optimisé pour grandes images

Early stopping : Prévention overfitting

## CONCLUSION TECHNIQUE
Le travail démontre une préparation rigoureuse des données médicales, avec une attention particulière aux spécificités du domaine :

Nettoyage robuste des données cliniques
Analyse exploratoire complète révélant la complexité des données
Préparation optimisée pour le deep learning
Identification proactive des défis techniques

La fondation établie permet le développement de modèles de classification robustes pour l'assistance au diagnostic radiologique.




