#!/bin/bash
TERM=xterm-256color
COLUMNS=238
prependWidth=26
tail=5

# Formatting
reset=`echo -e '\033[0m'`
black=`echo -e '\033[0;30m'`
red=`echo -e '\033[1;31m'`
green=`echo -e '\033[1;32m'`
yellow=`echo -e '\033[1;33m'`
blue=`echo -e '\033[1;34m'`
purple=`echo -e '\033[1;35m'`
cyan=`echo -e '\033[1;36m'`
gray=`echo -e '\033[0;37m'`

function prepend_output () {
    outputtingService=$1
    sed -e "s/^/${outputtingService} /"
}

function populate_namespaces_array () {
    x=1
    for line in $namespaces ;
    do
        if [ $line != "carama-int" ]
        then
            namespacesArray[x]=$line
        fi
        x=$((x+1))
    done
}

function kill_existing_port_forwarding () {
    echo -e "${red}Killing any existing port-forwarding processes...${reset}"
    echo ""
    $(killall kubectl port-forward &> /dev/null)
}

function monitoring_service () {
    while [ true ]; do
        adminServiceConnection
        blobServiceConnection
        [[ $frontend = "true" ]] && frontendServiceConnection
        taskManagerServiceConnection
        userServiceConnection
        workshopServiceConnection
    done
}

function adminServiceConnection () {
    if [[ ( -z "$adminServicePortForwardPID" ) || ( ! -z "$adminServicePortForwardPID" &&  ! $(ps -o pid= -p $adminServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-admin-api $adminServicePort:80 -n $chosenNamespace | prepend_output $adminPortFwdSvcProcessName &
        adminServicePortForwardPID=$!
    fi
}

function blobServiceConnection () {
    if [[ ( -z "$blobServicePortForwardPID" ) || ( ! -z "$blobServicePortForwardPID" &&  ! $(ps -o pid= -p $blobServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-blob-storage $blobServicePort:80 -n $chosenNamespace | prepend_output $blobPortFwdSvcProcessName &
        blobServicePortForwardPID=$!
    fi
}

function frontendServiceConnection () {
    if [[ ( -z "$frontendServicePortForwardPID" ) || ( ! -z "$frontendServicePortForwardPID" &&  ! $(ps -o pid= -p $frontendServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-frontend $frontendServicePort:80 -n $chosenNamespace | prepend_output $frontendPortFwdSvcProcessName &
        frontendServicePortForwardPID=$!
    fi
}

function taskManagerServiceConnection () {
    if [[ ( -z "$taskManagerServicePortForwardPID" ) || ( ! -z "$taskManagerServicePortForwardPID" &&  ! $(ps -o pid= -p $taskManagerServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-taskmanager $taskManagerServicePort:80 -n $chosenNamespace | prepend_output $taskManagerPortFwdSvcProcessName &
        taskManagerServicePortForwardPID=$!
    fi
}

function workshopServiceConnection () {
    if [[ ( -z "$workshopServicePortForwardPID" ) || ( ! -z "$workshopServicePortForwardPID" &&  ! $(ps -o pid= -p $workshopServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-workshop $workshopServicePort:80 -n $chosenNamespace | prepend_output $workshopPortFwdSvcProcessName & 
        workshopServicePortForwardPID=$!
    fi
}

function userServiceConnection () {
    if [[ ( -z "$userServicePortForwardPID" ) || ( ! -z "$userServicePortForwardPID" &&  ! $(ps -o pid= -p $userServicePortForwardPID)  ) ]]
    then
        kubectl port-forward svc/carama-user $userServicePort:80 -n $chosenNamespace | prepend_output $userPortFwdSvcProcessName & 
        userServicePortForwardPID=$!
    fi
}

function open_in_chrome () {
    mkdir "First Run"
    /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --host-rules="MAP carama-admin-api 127.0.0.1:$adminServicePort,MAP carama-blob-storage 127.0.0.1:$blobServicePort,MAP carama-taskmanager 127.0.0.1:$taskManagerServicePort,MAP carama-user 127.0.0.1:$userServicePort,MAP carama-workshop 127.0.0.1:$workshopServicePort" --user-data-dir="./First Run" --no-first-run --disable-web-security --allow-file-access-from-files --allow-file-access http://localhost:3010 | prepend_output $frontendPortFwdSvcProcessName &
}

function cycle_through_services () {
    echo "Initiating trailing logs for all backend services in $chosenNamespace environment..." | prepend_output $systemProcessName
    for deployment in $(kubectl get deploy -o name --no-headers -n $chosenNamespace );
    do
        stream_remote_log $deployment &
    done
}

function stream_remote_log () {
    case $1 in
        *"carama-admin-api"*)
            outputColour=$red
            ;;
        *"carama-blob"*)
            outputColour=$purple
            ;;
        *"carama-taskmanager"*)
            outputColour=$green
            ;;
        *"carama-user"*)
            outputColour=$blue
            ;;
        *"carama-workshop"*)
            outputColour=$purple
            ;;
        *)  outputColour=
            ;;
    esac

    local serviceName=$(sed 's/deployment.extensions\//remote-/' <<< "$1")
    local serviceName=$serviceName
    local serviceName=$(tr '[:lower:]' '[:upper:]' <<< "$serviceName")
    # Pads out serviceName variable with "-" to make uniform length service names in logging output
    local serviceName=$(awk '{x=$0;for(i=length;i<24;i++)x=x "-";}END{print x}' <<< "$serviceName")
    local serviceName="$serviceName$reset"
    local serviceName=$(sed -e "s/^/${outputColour}/" <<< "$serviceName")
    kubectl logs $1 -n $chosenNamespace --tail=$tail -f | less -S +F | prepend_output $serviceName &
}

function start () {
    clear

    kill_existing_port_forwarding | prepend_output $systemProcessName
    echo ""
    echo ""
    echo "Initiating port-forwarding service..." | prepend_output $systemProcessName
    echo ""
    echo ""
    echo "------------------------------------------USE CTRL-C TO EXIT------------------------------------------"
    echo "                       Running server against '''$chosenNamespace''' namespace"
    echo "------------------------------------------------------------------------------------------------------"
    echo ""
    echo "                           AUTO-START FRONTEND = $autostart                                           "
    echo "                                    K8 LOGGING = $logging                                             "
    echo ""
    echo "                         CARAMA_GOOGLE_API_KEY = $CARAMA_PUBLIC_GOOGLE_API_KEY                        "
    echo "                                      NODE_ENV = $NODE_ENV                                            "
    echo ""
    echo ""
    echo ""
    echo "                              Frontend will open in chrome automatically                              "
    echo ""
    echo ""
    echo "                        Backend services are available at http://localhost on the                     "
    echo "                           following ports for the corresponding service                              "
    echo "                                 admin-service - http://carama-admin-api"
    echo "                                  blob-service - http://carama-blob-storage"
    echo "                           taskmanager-service - http://carama-taskmanager"
    echo "                                  user-service - http://carama-user"
    echo "                              workshop-service - http://carama-workshop"
    echo ""
    echo ""
    echo "------------------------------------------------------------------------------------------------------"
    echo ""
    echo ""
    
    open_in_chrome

    if [ $logging = "true" ]
    then
        cycle_through_services
    fi
    monitoring_service
}

function adjust_settings () {
    clear
    local choice
    read -n 1 -p "Logging is set tail $tail lines. Would you like to pull in ALL logs from deployment of pod? (y/n) " choice
    case $choice in
        [yY] ) tail=9999999 ;;
        [nN] ) tail=$tail ;;
        *) echo "invalid entry" ;;
    esac
    show_menus
}

function bring_it_all_down () {
    clear
    pkill -9 -g $parentPID && echo "ALL CHILD PROCESSES HAVE BEEN TERMINATED. ERASING TEMPORARY FILES..."
    rm -rf "./First Run"
    exit 0
}

function show_menus () {
	clear
    echo "--------------------------------------"
	echo "  C A R A M A    N A M E S P A C E S"
	echo "--------------------------------------"
	i=1
    for namespace in ${namespacesArray[@]} ;
    do
        echo $i - $namespace
        i=$((i+1))
    done
    echo ""
    echo "s - SETTINGS"
    echo "q - EXIT"
    echo ""
    echo ""
    read_options
}

function read_options () {
	local choice
	read -n 1 -p "Select namespace to run against " choice
    chosenNamespace="${namespacesArray[$choice]}"
	case $choice in
        $option) start ;;
        s) adjust_settings ;;
		q) bring_it_all_down ;;
		*) echo -e "${RED}Select a number from the list or press 'q' to exit${STD}" && sleep 2
	esac
}

# GLOBALS
parentPID=$(ps -p $$ -o pgid=)
logging="true"
frontend="true"

# Exit procedure
trap 'bring_it_all_down' SIGHUP SIGINT SIGQUIT SIGTERM SIGABRT

# Check command line args
while getopts nsqh option
do
    case "${option}" in
        n) chosenNamespace="carama-int" start;;
        s) frontend="false";;
        q) logging="false";;
        h) echo "-q disables K8 logging" && exit
    esac
done

# Service port declarations
adminServicePort="6001"
blobServicePort="6002"
frontendServicePort="3010"
taskManagerServicePort="6004"
userServicePort="6005"
workshopServicePort="6006"

# Process ID variable declarations
adminServicePortForwardPID=
blobServicePortForwardPID=
taskManagerServicePortForwardPID=
userServicePortForwardPID=
workshopServicePortForwardPID=

# Logging service names
envsProcessName="${yellow}ENVS--------------------${reset}"
systemProcessName="${yellow}SYSETM------------------"
yarnProcessName="${yellow}YARN--------------------${reset}"
adminPortFwdSvcProcessName="${green}PORTFWD-ADMIN-SVC-------${reset}"
blobPortFwdSvcProcessName="${green}PORTFWD-BLOB-SVC--------${reset}"
frontendPortFwdSvcProcessName="${white}PORTFWD-FRONTEND--------${reset}"
taskManagerPortFwdSvcProcessName="${green}PORTFWD-TASKMNGR-SVC----${reset}"
userPortFwdSvcProcessName="${green}PORTFWD-USER-SVC--------${reset}"
workshopPortFwdSvcProcessName="${green}PORTFWD-WORKSHOP-SVC----${reset}"

# Namespace array
namespaces=$(kubectl get ns | awk '{print $1}' | grep 'carama\|pr-')
namespacesArray=()
populate_namespaces_array
len=${#namespacesArray[@]}
option=[1-$len]

# Begin
show_menus
