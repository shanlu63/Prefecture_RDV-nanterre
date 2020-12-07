# Prefecture_RDV
# Python program for RDV searching


import datetime
import os
import requests
import sys
import time
import winsound

URL = "https://www.hauts-de-seine.gouv.fr/booking/create/11658/0"
# We need to look like a normal browser because otherwise we are blocked
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) ' \
     'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36'
}

checkPhrase = "Il n'existe plus de plage horaire libre pour votre " \
    "demande de rendez-vous."

thisDir = os.path.dirname(__file__)

def checkPlacesAreAlreadyTaken():
    with requests.Session() as s:
        r = s.get(URL, headers=headers)
        r = s.post(URL, headers=headers,
                   data={'condition':'on',
                         'nextButton':'Effectuer+une+demande+de+rendez-vous'})
    return checkPhrase in r.text

def writeResults(placesAreAlreadyTaken):
    """Log the result to STDOUT and to two different files."""
    now = datetime.datetime.now()
    print('{} All places are already taken: {}'.format(now, placesAreAlreadyTaken))
    if placesAreAlreadyTaken:
        with open(os.path.join(thisDir, 'placesAreAlreadyTaken.log'), 'a') as f:
            f.write('{} Places are taken\n'.format(now))
    else:
        with open(os.path.join(thisDir, 'placeIsAvailable.log'), 'a') as f:
            f.write('{} At least one place is available!\n'.format(now))

def playSound():
    """Emit three terminal beeps within 1.5 s."""
    for i in range(5):
        winsound.PlaySound("*", winsound.SND_ALIAS)
        time.sleep(2)


def main():
    print('Starting to monitor the reservation system.')
    print('Press CTRL+C to abort...\n')
    while True:
        try:
            placesAreAlreadyTaken = checkPlacesAreAlreadyTaken()
        except requests.exceptions.RequestException as e:
            sys.stderr.write('** Error while querying site: {}\n'.format(e))
            playSound()
        else:
            if not placesAreAlreadyTaken:
                playSound()
            writeResults(placesAreAlreadyTaken)
        time.sleep(60)

if __name__ == '__main__':
    sys.exit(main())
