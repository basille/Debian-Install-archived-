Debian
======

Debian stuff

Debian Testing (Jessie) post-install

*Table of Contents*
- [[#image-debian][Image Debian]]
- [[#systeme][Système]]
- [[#general][Général]]


** Image Debian

- Récupérer [image USB/HDD](http://cdimage.debian.org/debian-cd/current-live/amd64/usb-hdd/) : 
- Vérifier les sommes md5.
- Copier l'image sur une clé usb vierge (attention, ça efface tout !) :
    $ sudo dd if=debian-live-6.0.6-amd64-gnome-desktop.img of=/dev/sdX
    $ sudo dd if=debian-XXXXXX-live-rc1-amd64-gnome-desktop+nonfree.iso of=/dev/sdb

- Booter sur la clé !
- Lancer l'install et suivre les instructions...
> Swap = RAM × 2 si 4 Go max de RAM, sinon = RAM
    - Si besoin, ajout d'un [ficher de SWAP](http://www.linux.com/learn/tutorials/442430-increase-your-available-swap-space-with-a-swap-file)


** Système

• Au démarrage, installer sudo si besoin :
# adduser mathieu sudo
∘ Puis reloguer :
# su - mathieu

• Enlever le bip système :
∘ https://wiki.archlinux.org/index.php/Disable_PC_Speaker_Beep
∘ Ce qui a marché pour moi (Gnome 3.8) :
xset -b
‣ Mettre dans les applications au démarrage :
$ gnome-session-properties
Beep system OFF
xset -b
∘ Gnome 3.12 : Paramètres > Son > Effets sonores > Volume 0

• Modifier le bash : 
∘ Menu, couleurs, fond transparent via les options
∘ Autocomplétion en root + Ctrl + N garde le répertoire courant + Manpage en couleurs + Alias upgrade + Alias VPN UF + Alias wake-on-lan + Alias appli (trouver le nom d'une application) + Utilisation des alias avec sudo + Ctrl + page up/down recherche une complétion dans l'historique + screen (pour serveur)
$ sudo nano /etc/bash.bashrc
∘ Ajouter ou décommenter les lignes suivantes :
---------------------------------------------------------------------
if [ -f /etc/bash_completion ]; then
. /etc/bash_completion
fi
### Ctrl + N garde le répertoire courant
export PS1='\[$(__vte_ps1)\]'$PS1
### Manpages en couleurs
export MANPAGER="/usr/bin/most -s"
### Alias upgrade
alias upgrade='sudo aptitude update && sudo aptitude safe-upgrade && sudo aptitude clean && sudo aptitude autoclean && sudo aptitude purge ~c'
### Alias VPN UF
alias vpnuf='echo -n ">iNX9(b8,MpX" | sudo openconnect -u basille@ufl.edu --passwd-on-stdin https://vpn.ufl.edu/'
### Alias wake-on-lan
alias wol='wakeonlan -i 10.251.3.175 18:03:73:3c:cb:4d'
### Alias appli (trouver le nom d'une application)
alias appli='ps --no-header -o comm -p $(xwininfo -all | grep "Process id:" | cut -d":" -f2 | cut -d" " -f2)' 
### When using sudo, use alias expansion (otherwise sudo ignores your aliases)
alias sudo='sudo '
### Prevent screen to turn off (server only)
xset s off # Disable X windows screen saver
xset -dpms # Disable display power management system
### Bash history with Ctrl + page up/down
nano .inputrc
"\e[5~": history-search-backward
"\e[6~": history-search-forward
# "\e[A": history-search-backward
# "\e[B": history-search-forward
set show-all-if-ambiguous on
set completion-ignore-case on
"\e[1;5C": forward-word
"\e[1;5D": backward-word
---------------------------------------------------------------------

• Reloguer... puis désactiver le compte root si tout va bien :
$ sudo passwd -l root
∘ Pour réactiver le compte root :
$ sudo passwd -u root

• Puis installer le système complet et fonctionnel :
$ sudo tasksel install gnome-desktop --new-install
(ou $ sudo aptitude install gnome-desktop-environment
∘ Et pour finir :
$ sudo aptitude install gnome
∘ Si besoin:
$ sudo tasksel install desktop
$ sudo tasksel install laptop
$ sudo aptitude install gnome-session
$ sudo aptitude install gnome-terminal
$ sudo aptitude install gdm3

• Problème de luminosité ? 
∘ http://ihisham.com/2013/12/fix-gnome-screen-brightness-keys-bug/

• Régler la SWAP pour être utilisée quand 100% de la RAM est utilisée :
$ sudo nano /etc/sysctl.conf
∘ Ajouter : 
---------------------------------------------------------------------
# SWAP after 100% RAM used 
vm.swappiness = 0
---------------------------------------------------------------------
• Si besoin de basculer la SWAP sur la RAM (nécessite autant de RAM disponible que de SWAP utilisée) : 
$ sudo swapoff -av
$ sudo swapon -av

• Si pas fait pendant l'install, régler espace réservé pour root dans la partition home (passer de 5% à 3%) :
$ sudo tune2fs -m 3 /dev/XXX
∘ (où XXX est donné par 'df' et correspond au 'home') (pour vérifier -l)

• Automount de disques externes :
$ sudo nano /etc/fstab
∘ Commenter la ligne du disque USB.

• Optimiser le disque SSD
∘ https://wiki.debian.org/SSDOptimization
∘ Diminuer la fréquence d'écriture des partitions + Améliorer les performances + Placer /tmp dans la RAM
‣ http://doc.ubuntu-fr.org/ssd_solid_state_drive#diminuer_la_frequence_d_ecriture_des_partitions
$ sudo nano /etc/fstab
‣ Rajouter l'option 'noatime' pour chaque partition SSD
‣ Rajouter l'option 'discard' pour chaque partition SSD
# /tmp dans la RAM
tmpfs      /tmp            tmpfs        defaults,size=1g
‣ Mettre à jour les réglages :
$ sudo update-initramfs -u -k all
∘ Supprimer le fichier .xsession-errors
# echo 'ln -fs /dev/null "$HOME"/.xsession-errors' > /etc/X11/Xsession.d/00disable-xsession-errors

• Sources.list : 
∘ http://wiki.debian-facile.org/manuel:sources.list-df
∘ http://wiki.debian-facile.org/manuel:apt:pinning
$ sudo nano /etc/apt/sources.list

• Apt-pinning : 
∘ http://wiki.debian-facile.org/manuel:configuration:pinning#fichier_preferences_pour_etre_en_testing_avec_le_pinning_sur_stable_unstable_et_experimental
$ sudo nano /etc/apt/preferences

• Pour éviter d'avoir les index de traduction :
$ sudo nano /etc/apt/apt.conf.d/apt.conf
∘ Ajouter :
---------------------------------------------------------------------
Acquire::Languages "none";
---------------------------------------------------------------------

• Mise-à-jour et installation complète
$ sudo aptitude update
$ sudo aptitude install deb-multimedia-keyring
$ sudo aptitude install apt-listbugs
$ sudo apt-cache policy

• Puis commenter la ligne de apt.conf au-dessus (devrait ne télécharger que en/fr)
$ sudo aptitude update
$ sudo aptitude safe-upgrade
$ sudo aptitude full-upgrade
$ upgrade

• WIFI Firmware support (http://wiki.debian.org/fr/iwlwifi)
$ sudo aptitude install firmware-iwlwifi
$ sudo modprobe -r iwlwifi
$ sudo modprobe iwlwifi


Général

$ sudo aptitude install aspell aspell-fr aspell-en autoconf bijiben build-essential chromium-browser cmake cmake-curses-gui conky-all debian-goodies disper dosbox elinks epiphany-browser espeak firmware-linux-free flashplugin-nonfree gcstar gftp gir1.2-gweather-3.0 git gkbd-capplet gnome-shell-extensions gnome-tweak-tool gnote gparted gtg gtick gtk2-engines-pixbuf gvncviewer hibernate hunspell-en-ca hunspell-en-us hunspell-fr libreoffice-pdfimport marble most mozplugger myspell-en-gb network-manager-openconnect-gnome network-manager-vpnc-gnome ntp pandoc pandoc-citeproc python-vte revelation rsync screen stellarium subversion telepathy-haze terminator transmission tree ttf-mscorefonts-installer ttf-arphic-ukai ttf-arphic-uming ttf-arphic-gkai00mp ttf-arphic-gbsn00lp ttf-arphic-bkai00mp ttf-arphic-bsmi00lp ttf-kochi-gothic ttf-kochi-mincho ttf-baekmuk unetbootin unison units unrar vpnc wakeonlan yafc
Pour libreoffice 3.5 (actuellement 3.4) : libreoffice-gtk3
(icedtea6-plugin)
(nautilus-open-terminal)
(python-evolution)
(transmission-daemon)

Reporting tool for i3, i5, i7
sudo aptitude install i7z i7z-gui

Mozilla + web
sudo aptitude install iceweasel iceweasel-l10n-fr icedove icedove-l10n-fr iceowl-extension iceowl-l10n-fr torbrowser-launcher
* User agent de Icedove : 
Options > Avancé > Éditeur de configuration
Ajouter une chaine de caractères 'general.useragent.override' avec : Mozilla/5.0 (X11; Linux x86_64; rv:17.0) Gecko/17.0 Thunderbird/17.0
(le user agent normal étant : Mozilla/5.0 (X11; Linux x86_64; rv:17.0) Gecko/17.0 Icedove/17.0)
À mettre à jour à chaque nouvelle version...
* Calendrier
gsettings set org.gnome.desktop.default-applications.office.calendar exec icedove
Créer un faux compte sous Evolution ; puis Fichier > Nouveau > Calendrier ; Type : CalDAV, Nom : Agenda calDav, « Marquer comme calendrier par défaut », URL : caldav://mathieu.basille.net/cloud/remote.php/caldav/calendars/mathieu/default%20calendar/ (ou mettre caldav://mathieu.basille.net/cloud/remote.php/caldav/calendars/mathieu/ et rechercher les calendriers), Rafraichir aux 15 minutes, Appliquer. Fermer Evolution...
Intégration à Gnome :
* Thunderbird : https://github.com/gnome-integration-team/thunderbird-gnome
* Les deux : https://addons.mozilla.org/fr/firefox/addon/htitle/

Suppression des liens des dicos fr_*
$ sudo rm /usr/share/hunspell/fr_*
$ sudo rm /usr/share/myspell/dicts/fr_*
En cas de problème, réinstaller hunspell-fr


Images / photos / multimédia / jeux
$ sudo aptitude install gimp-gmic gimp-plugin-registry gimp-resynthesizer gthumb hugin imagemagick inkscape darktable rawtherapee phatch qtpfsgui cuetools easytag flac gstreamer1.0-ffmpeg gstreamer1.0-fluendo-mp3 gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly monkeys-audio shntool soundconverter devede easytag oggconvert pitivi frei0r-plugins gnome-video-effects-frei0r openshot rhythmbox-ampache sound-juicer sox subtitleeditor vlc vorbis-tools vorbisgain xbmc sweethome3d qarte chromium-bsu


Slowmo : http://slowmovideo.granjow.net/
Récupérer package for Ubuntu Raring
Dépendances :
$ sudo aptitude install build-essential cmake git ffmpeg libavformat-dev libavcodec-dev libswscale-dev libqt4-dev freeglut3-dev libglew1.5-dev libsdl1.2-dev libjpeg-dev libopencv-video-dev libopencv-highgui-dev
(qgis 2.0 time managerattention, conflit entre libopencv-highgui-dev qui demande libtiff4 alors que libtiff5 est installée...)
Puis
$ sudo dpkg -i slowmovideo_0.3.1-5~raring1_amd64.deb


QGIS, GEOS, GDAL, PROJ.4
$ sudo aptitude install libgdal-dev libgeos-dev gdal-bin qgis python-qgis libproj-dev proj-bin


R
$ sudo aptitude install r-base-core r-base-dev r-recommended r-cran-rodbc r-cran-tkrplot littler jags libcairo2-dev libglu1-mesa-dev libxt-dev

Copie des fichiers de config (.Renviron, .Rprofile, dossier .R-site)

Package list:
> install.packages(c("ade4", "adehabitat", "adehabitatHR", "adehabitatHS", "adehabitatLT", "adehabitatMA", "beanplot", "biomod2", "Cairo", "circular", "colorRamps", "coxme", "data.table", "devtools", "dismo", "dplyr", "foreign", "fortunes", "gam", "ggplot2", "knitcitations", "knitr", "lme4", "lubridate", "maptools", "markdown", "moments", "MuMIn", "plyr", "randomForest", "raster", "rasterVis", "RColorBrewer", "RCurl", "reshape2", "rgdal", "rgeos", "rms", "roxygen2", "RPostgreSQL", "rworldmap", "rworldxtra", "scales", "SDMTools", "sp", "spacetime", "stringr", "testthat", "trip", "XML"))

Après installation de GDAL/GEOS/PROJ.4 :
> install.packages(c("rgdal", "rgeos"))

Packages perso :
> install.packages(c("basr", "hab", "seasonality", "rpostgis"), repos = "http://ase-research.org/R/")
Ou version de dév :
> library(devtools)
> install_github("basille/basr")
> install_github("basille/hab")
> install_github("basille/seasonality")
> install_github("basille/rpostgis")


Emacs + LaTeX + pdf (biblatex est dans texlive-bibtex-extra qui vient avec texlive-full / pdfmanipulate vient avec calibre)
$ sudo aptitude install emacs24 ispell texlive-full bibtex2html rubber jabref latex2rtf xpdf pdftk pdfjam poppler-utils libtext-pdf-perl pdf2svg impressive pdfchain pdfshuffler calibre mupdf pdf2djvu scribus xournal
(emacs emacs-goodies-el ess org-mode)
(ocrfeeder ocrodjvu)

Police différente dans Emacs et gedit (par exemple) : gnome-tweak-tool > Polices > Optimisation > Full)
$ nano /home/mathieu/.local/share/applications/emacs.desktop
[Desktop Entry]
Version=1.0
Name=Emacs
GenericName=Text Editor
Comment=View and edit files
MimeType=text/english;text/plain;text/x-makefile;text/x-c++hdr;text/x-c++src;te$
Exec=/usr/bin/emacs %F
TryExec=emacs
Icon=/usr/share/icons/hicolor/scalable/apps/emacs.svg
Type=Application
Terminal=false
Categories=Utility;Development;TextEditor;

Installer un package perso (par exemple moderncv)
$ sudo nano /etc/texmf/texmf.d/03local.cnf
TEXMFHOME = ~/.emacs.d/texmf
$ sudo update-texmf
Pour vérifier :
$ kpsewhich --var-value TEXMFHOME
Puis placer les packages dans ~/.emacs.d/texmf/tex/latex/, terminer l'installation si besoin, e.g.:
$ latex moderntimeline.ins
$ latex moderntimeline.dtx
Placer les polices dans ~/.emacs.d/texmf/fonts/truetype/
Puis mettre à jour l'index TeX :
$ sudo texhash


Google Earth
The Debian way:
$ sudo aptitude install googleearth-package
$ sudo dpkg --add-architecture i386
$ sudo apt-get install alien ia32-libs-gtk lib32nss-mdns libfreeimage3 lsb-core msttcorefonts pax rpm ttf-dejavu ttf-bitstream-vera
$ make-googleearth-package --force
$ sudo dpkg -i googleearth*.deb
Mais ia32-libs impossible à installer... Solution : récupérer .deb officiel chez Google, puis :
$ dpkg-deb -R google-earth-stable_current_amd64.deb gg
Pour extraire les fichiers, aller dans DEBIAN et modifier Control (enlever la dépendance à ia32-libs), puis recréer l'archive :
$ dpkg-deb -b gg google-earth-stable_current_amd64_mod.deb
$ sudo dpkg -i google-earth-stable_current_amd64_mod.deb


Skype
http://wiki.debian.org/skype
$ sudo dpkg --add-architecture i386
$ sudo aptitude update
$ wget -O skype-install.deb http://www.skype.com/go/getskype-linux-deb
$ sudo dpkg -i skype-install.deb
$ sudo aptitude -f install
Si besoin, installer les dépendances à la main :
$ sudo aptitude install libc6:i386 libgcc1:i386 libqt4-dbus:i386 libqt4-network:i386 libqt4-xml:i386 libqtcore4:i386 libqtgui4:i386 libqtwebkit4:i386 libstdc++6:i386 libx11-6:i386 libxext6:i386 libxss1:i386 libxv1:i386 libssl1.0.0:i386 libpulse0:i386 libasound2-plugins:i386
Intégration DBus ? https://gist.github.com/nzjrs/1006316
Problème de son avec libpulse : https://bugs.archlinux.org/task/35690
$ sudo nano /usr/share/applications/skype.desktop
Remplacer :
Exec=skype %U
par
Exec=/usr/bin/env PULSE_LATENCY_MSEC=30 /usr/bin/skype %U


Adobe Reader (dans dmo)
$ sudo aptitude install acroread:i386




Evince comme visionneur par défaut sur le web :
# nano /etc/mozpluggerrc
Puis placer la ligne evince en tête des applications PDF


Virer Mono
$ sudo aptitude purge mono-runtime cli-common mono-4.0-gac


Francisation :
$ sudo dpkg-reconfigure locales
Choisir en-GB.UTF-8, en-US.UTF-8, fr-FR.UTF-8 (default), fr-CA.UTF-8
http://forum.hardware.fr/hfr/OSAlternatifs/debian-francisation-programmes-sujet_31606_1.htm
http://liberez-le-tux.servhome.org/blog/2011/04/22/franciser-un-systeme-debian/
http://wiki.debian.org/Locale
Si besoin, reconfigurer le dossier de bureau :
$ xdg-user-dirs-update --set DESKTOP /home/user/Bureau/
Pour vérifier :
$ less .config/user-dirs.dirs

Supprimer les locales inutiles
$ sudo aptitude install localepurge
$ sudo localepurge

Nettoyage final
$ upgrade


* Terminal

Personnalisation terminator (couleurs blanc sur noir, transparence 0.7, menu) ; terminator par défaut :
(pas exactement ça...)
$ sudo mv /usr/bin/gnome-terminal /usr/bin/gnome-terminal-gnome
$ sudo ln -s /usr/bin/terminator /usr/bin/gnome-terminal
Ouvrir un terminal dans Nautilus:
$ sudo aptitude install nautilus-actions
Importer le fichier Desktop suivant :
======  Ouvrir dans un Terminator  ===================
[Desktop Entry]
Type=Action
TargetLocation=true
ToolbarLabel[fr_FR]=Ouvrir dans un Terminator
ToolbarLabel[fr]=Ouvrir dans un Terminator
Name[fr_FR]=Ouvrir dans un Terminator
Name[fr]=Ouvrir dans un Terminator
Profiles=profile-zero;

[X-Action-Profile profile-zero]
MimeTypes=inode/directory;
Exec=terminator --working-directory=%f
Name[fr_FR]=Profil par défaut
Name[fr]=Profil par défaut
======================================================
Quelques insultes pour les erreurs de mots de passe :
	sudo visudo
Changer la ligne : 
	Defaults    env_reset,insults


* Nautilus

- Trier les dossiers avant les fichiers (l'option n'a pas d'effet) :
$ gsettings set org.gnome.nautilus.preferences sort-directories-first true
- Dossier des modèles :
$ touch /home/mathieu/Modèles/Texte\ brut
$ ln /home/mathieu/Work/templates/knitr-template.Rnw /home/mathieu/Modèles/Knitr.Rnw
$ ln /home/mathieu/Work/templates/rmarkdown-template.Rmd /home/mathieu/Modèles/RMarkdown.Rmd


* Système

- Régler les applications préférées (Menu perso > Paramètres système > Informations système > Applications par défaut)
- Date dans l'horloge : gsettings set org.gnome.desktop.interface clock-show-date true
- Raccourcis clavier (Basculer l'état d'agrandissement : Super+Entrée ; Client de messagerie : Super+E ; Navigateur Web : Super+W ; Dossier personnel : Super+H ; Masquer toutes les fenêtres normales : Super+D ; Verrouiller l'écran : Ctrl+Échap ; Raccourcis perso : Terminator : Super+T)
- Applications au démarrage :
(si besoin, créer le dossier : $ mkdir ~/.config/autostart )
* Ctrl droit pour accéder au menu contextuel : 
$ nano ~/.config/autostart/ctrl_r.desktop
[Desktop Entry]
Type=Application
Exec=xmodmap -e 'keycode 105 = Menu'
Hidden=false
X-GNOME-Autostart-enabled=true
Name=Ctrl droit pour accéder au menu contextuel
* Shift droit pour avoir le caractère supérieur (clavier US) :
$ nano ~/.config/autostart/shift_r.desktop
[Desktop Entry]
Type=Application
Exec=xmodmap -e 'keycode 62 = less greater'
Hidden=false
X-GNOME-Autostart-enabled=true
Name=Shift droit pour avoir le caractère supérieur (clavier US)
- Conserver l'activation du pavé numérique entre sessions :
$ gsettings set org.gnome.settings-daemon.peripherals.keyboard remember-numlock-state true


* Conky

$ nano ~/.conkyrc
### ===================== DÉBUT ===================== ###
use_xft yes
xftfont 123:size=8
xftalpha 0.1
total_run_times 0
own_window yes
own_window_type desktop
own_window_argb_visual yes
own_window_argb_value 255
own_window_transparent yes
own_window_hints undecorated,below,sticky,skip_taskbar,skip_pager
double_buffer yes
minimum_size 250 5
maximum_width 500
draw_shades no
draw_outline no
draw_borders no
draw_graph_borders no
default_color white
default_shade_color red
default_outline_color green
no_buffers yes
uppercase yes
cpu_avg_samples 2
net_avg_samples 1
override_utf8_locale yes
use_spacer left 

# Frequence de mise a jour (secondes)
update_interval 1

# Position en bas a droite
alignment bottom_right

# Decalage par rapport aux bordures
gap_x 30
gap_y 20

TEXT
${color EAEAEA}${font GE Inspira:pixelsize=55}${alignr}${time %H:%M}${font GE Inspira:pixelsize=18}
${voffset 10}${alignr}${color EAEAEA}${time %A} ${color D12122}${time %d} ${color EAEAEA}${time %B}
${font Ubuntu:pixelsize=10}${alignr}${color D12122}HD $color${fs_bar 7,150 /home}
${font Ubuntu:pixelsize=10}${alignr}${color D12122}RAM $color${membar 7,150}
${font Ubuntu:pixelsize=10}${alignr}${color D12122}SWAP $color${swapbar 7,150}
${font Ubuntu:pixelsize=10}${alignr}${color D12122}CPU $color${cpubar cpu1 7,36} $color${cpubar cpu2 7,35} $color${cpubar cpu3 7,35} $color${cpubar cpu4 7,35}
### ====================== FIN ====================== ###
Puis :
$ nano ~/.config/autostart/conky.desktop
[Desktop Entry]
Type=Application
Exec=conky
Hidden=false
X-GNOME-Autostart-enabled=true
Name=Conky
(pour relancer Conky :  killall -SIGUSR1 conky)


* Extensions Gnome

- Liste : https://extensions.gnome.org/local/
o Applications Menu
o Auto Move Windows
o Calculator
x Connection Manager
x Launch new instance
o Media player indicator
x Native Window Placement
o OpenWeather
o Panel World Clock
o Places Status Indicator
o Quick Close in Overview
x Removable Drive Menu
o Skype Integration
o Suspend Button
x SystemMonitor
x TopIcons
x User Themes
x Window List
o windowNavigator
x Workspace Indicator
- Not working for Gnome Shell 3.12
o Candy Thief
o Window options
o WindowOverlay Icons
o Workspace Navigator
o workspaceAltTab


* gFTP, Gnote, GTG

Copier les contenus des dossiers .gftp, .local/share/gnote et .local/share/gtg
Applications au démarrage : GTG (regarder dans les options) ; Gnotes :
$ nano ~/.config/autostart/gnote.desktop
[Desktop Entry]
Type=Application
Exec=/usr/bin/gnote %u
Hidden=false
X-GNOME-Autostart-enabled=true
Name=Gnote
Comment[fr_FR.UTF-8]=Prendre des notes, relier des idées, rester organisé


* R

$ mkdir ~/.R-site
$ mkdir ~/.R-site/site-library
$ cp .Renviron ~
$ cp .Rprofile ~
Copier le contenu de .R-site (sauf site-library)
Packages (après installation de GEOS & GDAL)
/!\ en 'sudo R' pour les avoir pour tous les utilisateurs...
> install.packages("adehabitatHS", dep = TRUE)
> install.packages(c("adehabitat", "rgdal", "raster"))
> install.packages(c("beanplot", "Cairo", "clusterSim", "ggplot2", "MuMIn", "lme4", "rms"))

Pour utiliser un plus haut niveau de la pile C, sous emacs : lancer un shell (M-x shell)
$ ulimit -s 30000
$ R
Associer le R : M-x ess-remote RET r RET


* Emacs

$ cp -R .emacs-site ~
$ cp .emacs ~
$ cp .xpdfrc ~
$ cp .Xresources ~
$ xrdb -merge ~/.Xresources


* JabRef

Importer préférences (PrefJabRef-2014-XX-XX)
Lier le répertoire de biblio à /home/mathieu/Work/biblio/PDF/
Pour avoir un aspect GTK, dans Options > Préférences > Avancé renseigner la classe avec "com.sun.java.swing.plaf.gtk.GTKLookAndFeel"
Mettre dans ~/.texmf-var/bibtex/ (créer le répertoire si besoin) un lien 'bib' vers le répertoire de biblio (/home/mathieu/Work/biblio/ par exemple)
$ mkdir ~/.texmf-var/
$ mkdir ~/.texmf-var/bibtex/
$ ln -s ~/Work/biblio/ ~/.texmf-var/bibtex/bib
Vérifier les dossiers de biblio avec: 
$ kpsewhich -show-path=.bib


* VPNC + SSH

Fichiers *.conf dans ~/.vpnc
En ligne de commande
# cp .vpnc/* /etc/vpnc/
# cd /etc/vpnc/
# ls -l
Ligne à vérifier pour ne passer que les .conf en 600
# chmod 600 *.conf
Sinon via network-manager, en installant network-manager-vpnc network-manager-vpnc-gnome

Copier .ssh/config
$ mkdir ~/.ssh
$ cp .ssh/config ~/.ssh/

Copier répertoire de scripts et unison :
$ cp -R .scripts ~
$ cp -R .unison ~
$ mkdir ~/.unison/bkp


Rockbox utility
Download Rockbox utility: http://www.rockbox.org/download/
Dézipper le fichier, puis copier RockboxUtility dans /usr/local/bin/
# mv RockboxUtility /usr/local/bin/rockbox
# chmod 755 /usr/local/bin/rockbox 
Thème Ambiance (activer les icones)


Ajouter un logiciel dans la liste Ouvrir avec...
- First look for the program (.desktop) in /usr/share/applications.
- Edit the program file so that the Exec line looks like:
Exec=yourprogram %U
- Now the program should show up in application list 


Fichiers RAW

## DCRAW 9.16 (version courante)
sudo aptitude install libjasper-dev libjpeg8-dev liblcms1-dev liblcms2-dev
sudo ldconfig
mkdir dcraw
cd dcraw
wget http://www.cybercom.net/~dcoffin/dcraw/dcraw.c
gcc -o dcraw -O4 dcraw.c -lm -ljasper -ljpeg -llcms
sudo mv dcraw /usr/bin
cd ..
rm -R dcraw

## Vignettes
sudo aptitude install ufraw ufraw-batch gimp-dcraw
sudo nano /usr/share/thumbnailers/raw.thumbnailer

[Thumbnailer Entry]
Exec=/usr/bin/ufraw-batch --embedded-image --out-type=png --size=%s %u --overwrite --silent --output=%o
MimeType=image/x-3fr;image/x-adobe-dng;image/x-arw;image/x-bay;image/x-canon-cr2;image/x-canon-crw;image/x-cap;image/x-cr2;image/x-crw;image/x-dcr;image/x-dcraw;image/x-dcs;image/x-dng;image/x-drf;image/x-eip;image/x-erf;image/x-fff;image/x-fuji-raf;image/x-iiq;image/x-k25;image/x-kdc;image/x-mef;image/x-minolta-mrw;image/x-mos;image/x-mrw;image/x-nef;image/x-nikon-nef;image/x-nrw;image/x-olympus-orf;image/x-orf;image/x-panasonic-raw;image /x-pef;image/x-pentax-pef;image/x-ptx;image/x-pxn;image/x-r3d;image/x-raf;image/x-raw;image/x-rw2;image/x-rwl;image/x-rwz;image/x-sigma-x3f;image/x-sony-arw;image/x-sony-sr2;image/x-sony-srf;image/x-sr2;image/x-srf;image/x-x3f;



### To do :

### Lieux (Québec, Lyon, Trondheim) --> météo OK, mais pas différents lieux :(

### sudo

### Clés SSH et GPG

### RSync
> Copier RSync dans .scripts/RSync
> Raccourci bureau vers les 2 avec les icones dans .scripts/Icones

### GCStar
