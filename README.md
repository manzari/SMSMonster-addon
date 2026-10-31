# SMS Monster add-on repository

Home Assistant add-on repository for [SMS Monster](https://git.hauptstadt.monster/manzari/SMSMonster), an SMS gateway
for SIM800C USB modems.

Add it in Home Assistant: *Settings → Add-ons → Add-on store → ⋮ → Repositories*, paste
`https://git.hauptstadt.monster/manzari/SMSMonster-addon`, then install **SMS Monster**.

This repository holds only the add-on metadata. Images are built from the main repository and published to
`git.hauptstadt.monster/manzari/{arch}-smsmonster`; `scripts/release-addon.sh` over there bumps the version here.
