longueur_corde = 10  # en mètres
poids_grimpeur = 70  # en kilogrammes
hauteur_chute = 5  # en mètres
type_assurage = "dynamique"  # ou "statique"

masse = poids_grimpeur
gravité = 9.81
énergie_chute = masse * gravité * hauteur_chute

if type_assurage == "dynamique":
    facteur_reduction = 0.5
    énergie_impact = énergie_chute * facteur_reduction
else:
    énergie_impact = énergie_chute

print("Énergie de la chute :", énergie_chute, "Joules")
print("Énergie à l'impact :", énergie_impact, "Joules")
