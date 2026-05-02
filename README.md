# iMac 2009-2014 ulkoisena näyttönä virtanäppäintä painamalla

<img width="1560" height="720" alt="kuva" src="https://github.com/user-attachments/assets/878885d9-56f5-4789-b7d8-ab9c4307b1bb" />


_In english: [Use-iMac-2009-2014-as-an-external-display-in-Linux--macOS-and-Windows](https://github.com/Lumipyry/Use-iMac-2009-2014-as-an-external-display-in-Linux-macOS-and-Windows)_

Ohje soveltuu iMac vuosimalleihin 2009-2014. 

Hienointa on että kone voi vaihtaa ulkoiseksi näytöksi ja Linux koneeksi pelkästään virtanäppäintä painamalla - tai asettaa käynnistymään suoraan 2-näytöksi.

Pääkoneessa voi olla macOS, Linux tai Windows. 

Tämä ohje perustuu [smc_util](https://github.com/floe/smc_util):iin sisältäen FreekMank:n [tuen virtanäppäimen käytölle](https://github.com/floe/smc_util/pull/11/commits). Ulkoisena näyttönä käytettävään koneeseen tulee asentaa sekä Linux että Apple High Sierra tai aikaisempi.

Toimii todistettavasti MiniDisplay Port kaapelilla. Ei toimi Thunderbolt kaapelilla ainakaan jos näyttö-iMac ei tue sitä (2009-2011). Testattu iMac malleilla 2009-2011. Ei varmuutta Thunderbolt kaapelin toimivuudesta myöhemmissä iMac-malleissa (kokeilemalla selviää).

Jos pääkoneessa on macOS, on mahdollista käyttää myös macOS:n omaa näytön peilausta/ulkoista näyttöä asentamalla macOS Sequoia käyttäen [Open Core Legacy Patcher](https://github.com/dortania/Opencore-Legacy-Patcher)-ohjelmaa. Hyvä video sen asentamiseen on [täällä](https://www.youtube.com/watch?v=in5-3EjKFqA). Itse sain näytön peilauksen/ulkoisen näytön toimimaan vuoden 2011 21" iMacissa asentamalla Sequoian mutta en vuoden 2010 27" mallissa. Toisaalta se ei toiminut 2010 27" mallissa.
***
Luultavasti haluat jättää kohdan 16 (`rc.local`) pois asennuksesta - jos ja kun haluat myös käyttää näyttökoneen Linux käyttöjärjestelmää. (`rc.local` käynnistää koneen suoraan kohdenäyttötilaan - ja silloin virtanäppäimen painaminen voi johtaa tyhjään näyttöön). Ilman `rc.local`:a kone käynnistyy ensin Linuxissa.
***
Asennus tapahtuu seuraavilla komennoilla käyttämällä päätettä (Terminal)

1.Lataa smc_util 2-näyttökoneen kotihakemistoon
```
git clone https://github.com/floe/smc_util.git
```
2.Siirry hakemistoon smc_util
```
cd smc_util
```
3.Käännä SmcDumpkey GCC:llä
```
gcc -O2 -o SmcDumpKey SmcDumpKey.c -Wall
```
4.Luo tiedostot tdm_toggle.sh, powerbutton, powerbutton.sh ja rc.local
```
touch tdm_toggle.sh powerbutton powerbutton.sh rc.local
```
5.Vaihda tiedoston tdm_off.sh sisältö seuraavanlaiseksi (käyttämällä päätteessä komentoa `nano`) 
```
nano tdm_off.sh
```
tai avaamalla tiedosto tekstinkäsittelyohjelmalla 
```
#!/bin/bash
rm -f tdm_started
./SmcDumpKey MVHR 0
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --auto

```
6.Vaihda tiedoston tdm_on.sh sisältö seuraavanlaiseksi
```
nano tdm_on.sh
```
```
#!/bin/bash
./SmcDumpKey MVHR 1
sleep 1
./SmcDumpKey MVMR 2
sleep 2
DISPLAY=:0.0 xrandr --output eDP --off
touch tdm_started
```
7.Luo seuraava sisältö tiedostoon tdm_toggle.sh
```
nano tdm_toggle.sh
```
```
#!/bin/bash
pushd $(dirname "${BASH_SOURCE[0]}")

if [[ -f "tdm_started" ]]; then
  echo "Switching off TDM"
  source tdm_off.sh
else
  echo "Switch on TDM"
  source tdm_on.sh
fi

popd

```
8.Luo seuraava sisältö tiedostoon powerbutton
```
nano powerbutton
```
```
event=button/power PBTN
action=/etc/acpi/powerbutton.sh
```
9.Luo seuraava sisältö tiedostoon powerbutton.sh. `MUISTA VAIHTAA XXXXXXXXX omaan käyttäjänimeesi (KAHDESSA kohdassa scriptissä)`
```
nano powerbutton.sh
```
```
#!/bin/bash

pushd $(dirname "${BASH_SOURCE[0]}")

FILE=powerbutton_pressed
NOW=$(date +%s)

if [[ -f "$FILE" ]]; then
  # Read timestamp of previous powerbutton press from file
  echo "File exists"
  typeset -i PREV=$(cat $FILE)
  echo Compare $NOW and $PREV

  # if two powerbutton presses were <1 seconds apart -> shutdown
  if [[ $(($NOW-$PREV)) -lt 2 ]]; then
    # Shutdown
    echo "Powerbutton pressed twice in a row: Shutting down"
    shutdown now
  else
    echo "Toggle TDM"
    /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
  fi
else
  echo "Toggle TDM"
  /home/XXXXXXXXX/smc_util/tdm_toggle.sh &
fi

echo $NOW > $FILE

popd
```
10.Luo seuraava sisältö tiedostoon rc.local. `MUISTA VAIHTAA XXXXXXXXX omaan käyttäjänimeesi`
```
nano rc.local
```
```
#!/bin/bash

# Start Target Display Mode such that the pc is used as external monitor right away
# Note: The Power button toggles TDM (see /etc/acpi/events)
pushd /home/XXXXXXXXX/smc_util
./tdm_on.sh
popd
```
11.Poista SMC kernel ajuri konfliktien välttämiseksi
```
sudo rmmod applesmc
```
12.Muuta tdm_toggle.sh, rc.local ja powerbutton.sh suoritettaviksi (`vaihda XXXXXXXX`)
```
sudo chmod +x /home/XXXXXXXXX/smc_util/tdm_toggle.sh /home/XXXXXXXXX/smc_util/rc.local /home/XXXXXXXXX/smc_util/powerbutton.sh
```
13.Asenna build-essential ja acpid
```
sudo apt install build-essential acpid
```
14. Kopioi tiedosto powerbutton hakemistoon /etc/acpi/events (`vaihda XXXXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton /etc/acpi/events
```
15.Kopioi tiedosto powerbutton.sh hakemistoon /etc/acpi (`vaihda XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/powerbutton.sh /etc/acpi
```
16.Kopioi tiedosto rc.local hakemistoon /etc (`vaihda XXXXXXXX`)
```
sudo cp /home/XXXXXXXXX/smc_util/rc.local /etc/rc.local
```
17.Käynnistä `acpid` uudelleen
```
sudo systemctl restart acpid
```
18.Vaihda virtanäppäimen toiminta asetusten osassa Virranhallinta seuraavanlaiseksi

`Älä tee mitään`
***
TDM:n sammutus eli "Off": Kytke ensin ulkoinen näyttö pois käytöstä pääkoneen näyttöasetuksista ja paina sitten virtanäppäintä näyttönä toimivassa iMacissa.

***
Vinkki: Näyttönä toimivaan iMaciin kannattaa asentaa Samba palvelin ja kytkeä koneet yhteen Ethernet-piuhalla. Näin voit käyttää näyttökoneen tiedostojärjestelmää tallennustilana ja siirtää tiedostoja koneiden välillä. [Ohje löytyy täältä](https://github.com/Lumipyry/Luo-kotiverkko-ja-asenna-Samba-palvelin)
***
