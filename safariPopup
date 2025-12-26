#!/bin/zsh

#################################################################################################
#    Safari Pop-up Configuration Script
#    This script allows or blocks pop-ups for specified domains in Safari
#    based on parameters provided via Jamf Pro.                                                               
#    Created by Thameem on 26 Dec 2025
#    Before deploying in Production try in testing environment  
#   use the parameters in jamf pro 4 and 5   
#                                                   
##   Jamf Pro Parameters:
# $4 = Comma-separated list of domains to ALLOW (e.g., portal.raven.io, google.com)
# $5 = Comma-separated list of domains to BLOCK
#################################################################################################

ALLOW_LIST="$4"
BLOCK_LIST="$5"

# Get current logged-in user
loggedInUser=$(stat -f%Su /dev/console)
echo "Logged in user: $loggedInUser"

# Set path to the Safari Per-Site Preferences database
SAFARI_DB="/Users/$loggedInUser/Library/Safari/PerSitePreferences.db"

# Quit Safari if running to prevent DB lock
if pgrep -x Safari >/dev/null; then
    echo "Closing Safari..."
    pkill -x Safari
    sleep 2
fi

# Function to update the SQLite database
set_popup_rule() {
    local domain=$1
    local value=$2
    
    if [[ -z "$domain" ]]; then return; fi
    
    # Check if DB exists
    if [[ ! -f "$SAFARI_DB" ]]; then
        echo "Safari database not found at $SAFARI_DB. Skipping $domain."
        return
    fi

    # Value 2 = Allow, Value 0 = Block
    sudo -u "$loggedInUser" sqlite3 "$SAFARI_DB" \
    "INSERT OR REPLACE INTO preference_values (domain, preference, preference_value) \
    VALUES ('$domain', 'PerSitePreferencesPopUpWindow', '$value');"
}

# Process ALLOW list ($4)
if [[ -n "$ALLOW_LIST" ]]; then
    # Zsh handles splitting string into array via (s:,:)
    for domain in ${(s:,:)ALLOW_LIST}; do
        domain=$(echo $domain | xargs) # Trim whitespace
        echo "Allowing pop-ups for: $domain"
        set_popup_rule "$domain" 2
    done
fi

# Process BLOCK list ($5)
if [[ -n "$BLOCK_LIST" ]]; then
    for domain in ${(s:,:)BLOCK_LIST}; do
        domain=$(echo $domain | xargs) # Trim whitespace
        echo "Blocking pop-ups for: $domain"
        set_popup_rule "$domain" 0
    done
fi

echo "Configuration complete. Safari will update preferences on next launch."
