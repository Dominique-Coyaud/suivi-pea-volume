import yfinance as yf
from datetime import datetime, timedelta
import pandas as pd

def analyser_volumes_pea(symboles_pea):
    """
    Analyse les volumes de transaction pour les valeurs PEA
    
    Args:
        symboles_pea (list): Liste des symboles des actions PEA
    
    Returns:
        list: Liste des actions avec volume anormalement élevé
    """
    resultats = []
    
    for symbole in symboles_pea:
        try:
            # Créer l'objet ticker
            ticker = yf.Ticker(symbole)
            
            # Récupérer les données de marché sur 30 jours (1 mois)
            historique = ticker.history(period='1mo', interval='1d').reset_index()
            
            # Vérifier si les données sont disponibles
            if historique.empty or len(historique) < 5:
                print(f"Données insuffisantes pour {symbole}")
                continue
            
            # Calculer le volume moyen (exclure les volumes nuls)
            volumes_valides = historique[historique['Volume'] > 0]['Volume']
            if volumes_valides.empty:
                print(f"Aucun volume valide pour {symbole}")
                continue
                
            volume_moyen = volumes_valides.mean()
            dernier_volume = historique.iloc[-1]['Volume']
            
            # Vérifier si le dernier volume est valide
            if dernier_volume <= 0:
                print(f"Volume du jour invalide pour {symbole}")
                continue
            
            # Si le volume du jour est supérieur de 50% à la moyenne mensuelle
            if dernier_volume > 1.5 * volume_moyen:
                # Récupérer les informations supplémentaires
                info = ticker.info
                cours_cloture = historique.iloc[-1]['Close']
                
                # Récupérer les données sur 52 semaines pour les min/max
                historique_52sem = ticker.history(period='1y', interval='1d')
                if not historique_52sem.empty:
                    haut_52sem = historique_52sem['High'].max()
                    bas_52sem = historique_52sem['Low'].min()
                else:
                    # Fallback sur les données de l'API si disponibles
                    haut_52sem = info.get('fiftyTwoWeekHigh', 'N/A')
                    bas_52sem = info.get('fiftyTwoWeekLow', 'N/A')
                
                # Calculer le ratio d'augmentation du volume
                ratio_volume = dernier_volume / volume_moyen
                
                # Récupérer le nom de l'entreprise
                nom_entreprise = info.get('longName', info.get('shortName', 'N/A'))
                
                resultats.append({
                    'Date': datetime.today().strftime('%Y-%m-%d'),
                    'Symbole': symbole,
                    'Nom': nom_entreprise,
                    'Clôture (€)': round(cours_cloture, 2) if cours_cloture else 'N/A',
                    'Volume jour': int(dernier_volume),
                    'Volume moyen 30j': int(volume_moyen),
                    'Ratio volume': round(ratio_volume, 2),
                    'Bas 52 semaines': round(bas_52sem, 2) if isinstance(bas_52sem, (int, float)) else bas_52sem,
                    'Haut 52 semaines': round(haut_52sem, 2) if isinstance(haut_52sem, (int, float)) else haut_52sem
                })
                
        except Exception as e:
            print(f"Erreur avec {symbole} : {e}")
            continue
    
    return resultats

def afficher_resultats(resultats):
    """
    Affiche les résultats sous forme de tableau
    """
    if not resultats:
        print("Aucune action avec volume anormalement élevé trouvée.")
        return
    
    # Créer un DataFrame pour un affichage propre
    df = pd.DataFrame(resultats)
    
    # Trier par ratio de volume décroissant
    df = df.sort_values('Ratio volume', ascending=False)
    
    print(f"\n📊 Actions PEA avec volume anormalement élevé ({len(resultats)} trouvées)")
    print("=" * 100)
    print(df.to_string(index=False))

# Exemple d'utilisation
if __name__ == "__main__":
    # Liste d'exemple d'actions PEA (à adapter selon vos besoins)
    symboles_pea_exemple = [
        'MC.PA',      # LVMH
        'OR.PA',      # L'Oréal
        'SAP.PA',     # Sanofi
        'TTE.PA',     # TotalEnergies
        'BNP.PA',     # BNP Paribas
        'ATO.PA',     # Atos
        'CA.PA',      # Carrefour
        'RNO.PA',     # Renault
        'AIR.PA',     # Airbus
        'CAP.PA'      # Capgemini
    ]
    
    # Analyser les volumes
    resultats = analyser_volumes_pea(symboles_pea_exemple)
    
    # Afficher les résultats
    afficher_resultats(resultats)
