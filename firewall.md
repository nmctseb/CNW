# Firewall
ufw [--dry-run] enable|disable|reload

ufw [--dry-run] default allow|deny|reject [incoming|outgoing|routed]

ufw [--dry-run] logging on|off|LEVEL

ufw [--dry-run] reset

ufw [--dry-run] status [verbose|numbered]

ufw [--dry-run] show REPORT

ufw  [--dry-run] [delete] [insert NUM] allow|deny|reject|limit [in|out] [log|log-all] [ PORT[/PROTOCOL] | APPNAME ] [comment COMMENT]

ufw [--dry-run] [rule] [delete] [insert NUM] allow|deny|reject|limit [in|out [on INTERFACE]] [log|log-all] [proto  PROTO‐
COL] [from ADDRESS [port PORT | app APPNAME ]] [to ADDRESS [port PORT | app APPNAME ]] [comment COMMENT]

ufw  [--dry-run] route [delete] [insert NUM] allow|deny|reject|limit [in|out on INTERFACE] [log|log-all] [proto PROTOCOL]
[from ADDRESS [port PORT | app APPNAME]] [to ADDRESS [port PORT | app APPNAME]] [comment COMMENT]

ufw [--dry-run] delete NUM

ufw [--dry-run] app list|info|default|update
