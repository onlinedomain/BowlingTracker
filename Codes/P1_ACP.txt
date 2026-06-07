import numpy as np
import pandas as ps
import matplotlib.pyplot as plt

#%% Création de la fonction qui réalise l'ACP
def ACP(R):
    """
    Déroulement de l'ACP
        - Centrage (Manière matricielle)
        - Réduction (Manière matricielle)
        - Calcul de la SVD
        - Calcul de la matrice P
    
    Entrée
        R : Matrice non normalisée sur laquelle réaliser l'ACP
    
    Sortie
        P : Matrice des composantes principales
        Epsilon : Vecteur des valeurs singulières de la SVD, permet de
                  calculer le pourcentage d'information contenu dans les
                  n permières composantes principales
    """
    
    # Récupération de la taille de la matrice
    m,n = np.shape(R)
    
    # Centrage de la matrice
    R = R - 1/m*np.ones((m,1))@np.ones((1,m))@R

    # Réduction de la matrice
    variable = []
    for i in range(n):
        variable.append(np.sqrt(1/m*sum(R[:,i]**2)))
    D = np.diag(variable)
    R = R@np.linalg.inv(D)

    # SVD de la matrice centrée-réduite
    U,Epsilon,VT = np.linalg.svd(R)

    # Calcul de la matrice P
    P = R@VT.T
    
    return P, Epsilon

#%% Importation, traitement des données et mise sous forme de matrice

# Définition du chemin vers les données (enregistrées en csv)
chemin_donnees = "donnees_acp_avec_profils.csv"

# Importation des données via Pandas
df = ps.read_csv("Data/" + chemin_donnees)

# Sauvegarde des différents profils avant suppression de ceux-ci pour
# afficher des points de différentes couleurs par la suite
Profils = df["Profil"]

# Suppression des colonnes non pertinentes pour la comparaison (profil et ID)
df = df.drop('Profil', axis=1)
df = df.drop('ID', axis=1)

# Mise sous forme de matrice
R = df.to_numpy()

# Execution de l'ACP sur R
P, eps = ACP(R)

# Calcul du pourcentage d'information conservée avec seulement deux composantes 
taux_info = int(((eps[0]**2 + eps[1]**2)/sum(eps**2) )* 100)

# Dictionnaire des couleurs en fonction des profils
couleurs_des_points = {
    "Étudiant" : "blue",
    "Jeune sportif" : "red",
    "Parent" : "green",
    "Cadre senior" : "black"
    }

# Affichage des différentes couleurs pour chaque profil
for profil, couleur in couleurs_des_points.items():
    masque = profil == Profils
    
    # Ajout des points sur le graphique
    plt.scatter(P[masque,0], P[masque,1], c = couleur)
    
# Paramètres du graphique (titres et légendes)
plt.title("Répartition des individus en fonction des deux premières composantes principales")
plt.legend([i for i in couleurs_des_points.keys()], loc="upper left", 
           bbox_to_anchor=(1, 0.75), title=f"Taux d'infomation conservée : \n {taux_info}%")
plt.xlabel("Composante 1")
plt.ylabel("Composante 2")
plt.grid(visible="True")
plt.show()