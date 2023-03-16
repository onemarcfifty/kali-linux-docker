#!/bin/bash

# Import configuration variables from env file

export $(grep -v '^#' env | xargs)

# ##########################################################################
# Remote access variables (can be adapted)
# ##########################################################################
# set the default values for remote access ports
# ##########################################################################

if [ -z $XRDP_PORT ];
then
    XRDP_PORT=13389
fi

if [ -z $XVNC_DISPLAY ];
then
    XVNC_DISPLAY=8
fi

if [ -z $XVNC_PORT ];
then
    XVNC_PORT=590$XVNC_DISPLAY
fi

if [ -z $XSSH_PORT ];
then
    XSSH_PORT=20022
fi

# ##########################################################################
# Variables (For menu choice - should not be changed)
# ##########################################################################
# The following variables contain the default and the possible choices
# for Desktop Environment, Remote Access, Kali Packages, Network, and build
# architecture
# ##########################################################################

XDESKTOP_CHOICE=("xfce" "mate" "kde" "e17" "gnome" "i3" "i3-gaps" "live" "lxde")
XREMOTE_CHOICE=("vnc" "rdp" "x2go")
XKALI_CHOICE=("arm" "core" "default" "everything" "firmware" "headless" "labs" "large" "nethunter")
XNET_CHOICE=("bridge" "host")
XBUILD_CHOICE=("amd64" "arm64")

# ##########################################################################
# menu function
# ##########################################################################
# Takes an array as parameter, e.g. ("Color" "blue" "red")
# it will then prompt "Please select a xxx" and replace xxx with the first 
# element of the array (in the above example "Please select a color")
# It then shows numbers starting from 1 and options you may chose
# ( 1 - blue, 2 - red in the example) and return the user's choice
# in the variable $choice
# ##########################################################################


menu() {
        choice=0

        while true; do
                XCHOICE=("$@")
                echo -e "\nPlease select a $1"
                for (( i=1; i < ${#XCHOICE[@]} ; i++)) ; do
                        echo "$i - ${XCHOICE[$i]}"
                done
                read -n 1 -p "Your choice -> " choice
                echo -e 
                if (($choice >=1 && $choice < ${#XCHOICE[@]})); then  break ; fi
        done
}

# ##########################################################################
# Script starting point
# ##########################################################################

echo "This script will create a custom Kali Linux Docker container for you"

# ##########################################
# ask the user what he/she wants to install 
# we call the menu function with each of the 
# above variables if not specified in .env
# ##########################################

if [ -z $XDESKTOP_ENVIRONMENT ];
then
    menu  "Desktop Environment (only xfce for xrdp right now)" ${XDESKTOP_CHOICE[@]}
    XDESKTOP_ENVIRONMENT=${XDESKTOP_CHOICE[$choice-1]}
fi

if [ -z $XREMOTE_ACCESS ];
then
    menu  "Remote Access Option" ${XREMOTE_CHOICE[@]}
    XREMOTE_ACCESS=${XREMOTE_CHOICE[$choice-1]}
fi

if [ -z $XKALI_PKG ];
then
    menu  "Kali Package" ${XKALI_CHOICE[@]}
    XKALI_PKG=${XKALI_CHOICE[$choice-1]}
fi

if [ -z $XNETWORK ];
then
    menu  "Network" ${XNET_CHOICE[@]}
    XNETWORK=${XNET_CHOICE[$choice-1]}
fi

if [ -z $XBUILD_PLATFORM ];
then
    menu "Build Architecture" ${XBUILD_CHOICE[@]}
    XBUILD_PLATFORM=${XBUILD_CHOICE[$choice-1]}
fi

# ##########################################
# additional user config input:
# - Name for local custom Kali Docker image
# - Name for created container
# - Dir of host machine to mount
# - Dir to mount in container
# - Username for container user
# - Password for conatiner user (echo off)
# ##########################################

if [ -z $DOCKERIMG ];
then
    printf "Enter desired local Docker image name (e.g. custom/kali-linux): "
    read DOCKERIMG
    printf "\n"
fi

if [ -z $CONTAINER ];
then
    printf "Enter desired local Docker container name (e.g. kali-linux): "
    read CONTAINER
    printf "\n"
fi

if [ -z $HOSTDIR ];
then
    printf "Enter host directory to mount: "
    read HOSTDIR
    printf "\n"
fi

if [ -z $CONTAINERDIR ];
then
    printf "Enter container directory to mount to: "
    read CONTAINERDIR
    printf "\n"
fi

if [ -z $USERNAME ];
then
    printf "Enter desired username: "
    read USERNAME
    printf "\n"
fi

if [ -z $PASSWORD ];
then
stty -echo
    printf "Enter desired password (password will show in Docker output): "
    read PASSWORD
    stty echo
    printf "\n"
fi

# ##########################################
# show a summary of the Installation choices 
# and confirm choices
# ##########################################

clear
echo -e "Configuration:\n"
echo "Desktop environment:    $XDESKTOP_ENVIRONMENT"
echo "Remote Access:          $XREMOTE_ACCESS"
echo "Kali packages:          $XKALI_PKG"
echo -e "Network:                $XNETWORK"
echo -e "Build platform:         $XBUILD_PLATFORM"
echo -e "Image name:             $DOCKERIMG"
echo -e "Conatiner name:         $CONTAINER"
echo -e "Host dir mount:         $HOSTDIR"
echo -e "Container dir mount:    $CONTAINERDIR"
echo -e "Username:               $USERNAME"
echo -e "Password                [redacted]"

printf "\nHit enter to start building the container"
read

# ##########################################
# build the image
# ##########################################
# call docker build and pass on all
# the choices as build-arg to the Dockerfile
# where they will be interpreted
# ##########################################

docker image build --platform linux/$XBUILD_PLATFORM \
        -t $DOCKERIMG \
        --build-arg DESKTOP_ENVIRONMENT=$XDESKTOP_ENVIRONMENT \
        --build-arg REMOTE_ACCESS=$XREMOTE_ACCESS \
        --build-arg KALI_PACKAGE=$XKALI_PKG \
        --build-arg RDP_PORT=$XRDP_PORT \
        --build-arg VNC_PORT=$XVNC_PORT \
        --build-arg VNC_DISPLAY=$XVNC_DISPLAY \
        --build-arg SSH_PORT=$XSSH_PORT \
        --build-arg BUILD_ENV=$XBUILD_PLATFORM \
        --build-arg HOSTDIR \
        --build-arg CONTAINERDIR \
        --build-arg UNAME=$USERNAME \
        --build-arg UPASS=$PASSWORD \
        .

# ##########################################
# create the container
# ##########################################
# call docker create and pass on all
# the choices for network and ports that the
# user has made in the menu
# ##########################################

docker create   --name $CONTAINER \
                --network $XNETWORK \
                --platform linux/$XBUILD_PLATFORM \
                -p $XRDP_PORT:$XRDP_PORT \
                -p $XVNC_PORT:$XVNC_PORT \
                -p $XSSH_PORT:$XSSH_PORT \
                -t \
                -v $HOSTDIR:$CONTAINERDIR \
                $DOCKERIMG

# ##########################################
# start the container
# ##########################################

echo "Image ($DOCKERIMG) and container ($CONTAINER) build successful. $CONTAINER will now start."
docker start $CONTAINER

# Clear environment variables from env file

unset $(grep -v '^#' env | sed -E 's/(.*)=.*/\1/' | xargs)