#!/bin/bash
#
# Script used to sign applications via commandline
#

APK=$1
KEYSTORE=$2
ALIAS=$3

if [ -z $APK ] || [ -z $KEYSTORE ] || [ -z $ALIAS ]; then
    echo "All required info not provided"
    echo "path to apk, path to keystore, keystore alias"
    exit
fi

jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore $KEYSTORE $APK $ALIAS
