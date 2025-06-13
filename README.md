# suivi-pea-volume
Suivi quotidien des volumes de transactions sur les valeurs cotées en bourse éligibles au PEA. Valeurs dont le volume de transaction quotidien est anormalement élevé par rapport à sa moyenne mensuelle.
from yahooquery import Ticker
import pandas as pd
from datetime import datetime

# Liste simplifiée d'actions éligibles au PEA (extraits du CAC 40 ou SBF 120)
actions_pea = ['BNP.PA', 'AIR.PA', 'OR.PA', 'ENGI.PA', 'MC.PA']

# Résultats finaux
resultats = []

# Parcours de chaque action
for symbole in actions_pea:
    try:
        t = Ticker(symbole)

        # Récupérer les données de marché sur 30 jours (1 mois)
        historique = t.history(period='1mo', interval='1d').reset_index()
        if historique.empty:
            continue

        volume_moyen = historique['volume'].mean()
        dernier_volume = historique.iloc[-1]['volume']

        # Si le volume du jour est supérieur de 50 % à la moyenne mensuelle
        if dernier_volume > 1.5 * volume_moyen:
            info = t.summary_detail.get(symbole, {})
            cours_cloture = historique.iloc[-1]['close']
            haut_52sem = info.get('fiftyTwoWeekHigh')
            bas_52sem = info.get('fiftyTwoWeekLow')

            resultats.append({
                'Date': datetime.today().strftime('%Y-%m-%d'),
                'Symbole': symbole,
                'Clôture (€)': round(cours_cloture, 2),
                'Volume jour': int(dernier_volume),
                'Volume moyen 30j': int(volume_moyen),
                'Bas 52 semaines': bas_52sem,
                'Haut 52 semaines': haut_52sem
            })
    except Exception as e:
        print(f"Erreur avec {symbole} : {e}")

# Affichage dans un tableau
df = pd.DataFrame(resultats)

if not df.empty:
    print("Actions avec volume anormalement élevé :\n")
    print(df.to_string(index=False))
else:
    print("Aucune anomalie détectée aujourd'hui.")
