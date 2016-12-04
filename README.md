postfix
=======

Role which helps to install and configure Postfix.

The configuration of the role is done in such way that it should not be necessary
to change the role for any kind of configuration. All can be done either by
changing role parameters or by declaring completely new configuration as a
variable. That makes this role absolutely universal. See the examples below for
more details.

Please report any issues or send PR.


Example
-------

```
---

- name: Default usage
  hosts: host1
  roles:
    - postfix

- name: Customize main configuration
  hosts: host2
  vars:
    # Override some of the default configuration
    postfix_config_inet_interfaces: all
    postfix_config_inet_protocols: all

    # Add some custom configuration
    postfix_config__custom:
      myhostname: mydomain.com
      mydomain: mydomain.com
      myorigin: $mydomain
      mynetworks: 192.168.0.0/16
  roles:
    - postfix

- name: Pattern tables customization
  hosts: host3
  vars:
    # Add two records into the access table
    postfix_tables_access:
      1.2.3: REJECT
      1.2.3.4: OK

    # Add one records into the generic table
    postfix_tables_generic:
      his@localdomain.local: hisaccount@hisisp.example

    # Adding completely new table
    postfix_tables__custom:
      # This is the table file name
      my_table:
        # This is the content of the table
        first_pattern: first_result
        second_pattern: second_result
  roles:
    - postfix

- name: Master table customization
  hosts: host4
  vars:
    # Change max number of processes for the anvil process
    postfix_master_anvil_maxproc: 2

    # Set max number of processes for the showq process
    postfix_master_showq__custom:
      maxproc: 1

    # Add new process into the master table
    postfix_master__custom:
      - myprocess:
          type: unix
          chroot: no
          command: myprocess -o myopt=
  roles:
    - postfix
```


Role variables
--------------

List of variables used by the role:

```
# Package to be installed (explicit version can be specified here)
postfix_pkg: postfix

# Name of the Postfix service
postfix_service: postfix

# Command for postmap
postfix_postmap_cmd: postmap


# Location of the main Postfix config file
postfix_config_file: /etc/postfix/main.cf

# Paths
postfix_config_queue_directory: /var/spool/postfix
postfix_config_command_directory: /usr/sbin
postfix_config_daemon_directory: /usr/libexec/postfix
postfix_config_data_directory: /var/lib/postfix

# Mail owner
postfix_config_mail_owner: postfix

# Inet interface
postfix_config_inet_interfaces: localhost

# Inet protocols
postfix_config_inet_protocols: all

# Default mydestinations
postfix_config_mydestination__default:
  - $myhostname
  - localhost.$mydomain
  - localhost

# Custom mydestinations
postfix_config_mydestination__custom: []

# Final mydestinations
postfix_config_mydestination: "{{
  postfix_config_mydestination__default +
  postfix_config_mydestination__custom }}"

# Reject code
postfix_config_unknown_local_recipient_reject_code: 550

# Alias maps
postfix_config_alias_maps: hash:/etc/aliases

# Alias database
postfix_config_alias_database: hash:/etc/aliases

# Debug peer level
postfix_config_debug_peer_level: 2

# Default debugger command paths
postfix_config_debugger_command_path__default:
  - /bin
  - /usr/bin
  - /usr/local/bin
  - /usr/X11R6/bin

# Custom debugger command paths
postfix_config_debugger_command_path__custom: []

# Final debugger command paths
postfix_config_debugger_command_path: "{{
  postfix_config_debugger_command_path__default +
  postfix_config_debugger_command_path__custom }}"

# Debugger command
postfix_config_debugger_command: |2-
  
    PATH={{ postfix_config_debugger_command_path | join(':') }}
    ddd $daemon_directory/$process_name $process_id & sleep 5

# Bin paths
postfix_config_sendmail_path: /usr/sbin/sendmail.postfix
postfix_config_newaliases_path: /usr/bin/newaliases.postfix
postfix_config_mailq_path: /usr/bin/mailq.postfix

# Setgid group name
postfix_config_setgid_group: postdrop

# HTML directory (disabled by default)
postfix_config_html_directory: "no"

# Default Postfix configuration
postfix_config__default:
  queue_directory: "{{ postfix_config_queue_directory }}"
  command_directory: "{{ postfix_config_command_directory }}"
  daemon_directory: "{{ postfix_config_daemon_directory }}"
  data_directory: "{{ postfix_config_data_directory }}"
  mail_owner: "{{ postfix_config_mail_owner }}"
  inet_interfaces: "{{ postfix_config_inet_interfaces }}"
  inet_protocols: "{{ postfix_config_inet_protocols }}"
  mydestination: "{{ postfix_config_mydestination | join(', ') }}"
  unknown_local_recipient_reject_code: "{{ postfix_config_unknown_local_recipient_reject_code }}"
  alias_maps: "{{ postfix_config_alias_maps }}"
  alias_database: "{{ postfix_config_alias_database }}"
  debug_peer_level: "{{ postfix_config_debug_peer_level }}"
  debugger_command: "{{ postfix_config_debugger_command }}"
  sendmail_path: "{{ postfix_config_sendmail_path }}"
  newaliases_path: "{{ postfix_config_newaliases_path }}"
  mailq_path: "{{ postfix_config_mailq_path }}"
  setgid_group: "{{ postfix_config_setgid_group }}"
  html_directory: "{{ postfix_config_html_directory }}"

# Custom Postfix configuration
postfix_config__custom: {}

# Final Postfix config file
postfix_config: "{{
  postfix_config__default.update(postfix_config__custom) }}{{
  postfix_config__default }}"


# Access table
postfix_tables_access: {}

# Canonical table
postfix_tables_canonical: {}

# Generic table
postfix_tables_generic: {}

# Header checks table
postfix_tables_header_checks: {}

# Relocation table
postfix_tables_relocated: {}

# Transport table
postfix_tables_transport: {}

# Virtual table
postfix_tables_virtual: {}

# Default list of tables
postfix_tables__default:
  access: "{{ postfix_tables_access }}"
  canonical: "{{ postfix_tables_canonical }}"
  generic: "{{ postfix_tables_generic }}"
  header_checks: "{{ postfix_tables_header_checks }}"
  relocated: "{{ postfix_tables_relocated }}"
  transport: "{{ postfix_tables_transport }}"
  virtual: "{{ postfix_tables_virtual }}"

# Custom list of tables
postfix_tables__custom: {}

# Final list of tables
postfix_tables: "{{
  postfix_tables__default.update(postfix_tables__custom) }}{{
  postfix_tables__default }}"


# Location of the master.cf file
postfix_master_file: /etc/postfix/master.cf

# Default values of the anvil process configuration
postfix_master_anvil_type: unix
postfix_master_anvil_chroot: no
postfix_master_anvil_maxproc: 1
postfix_master_anvil_command: anvil

# Default anvil process configuration
postfix_master_anvil__default:
  type: "{{ postfix_master_anvil_type }}"
  chroot: "{{ postfix_master_anvil_chroot }}"
  maxproc: "{{ postfix_master_anvil_maxproc }}"
  command: "{{ postfix_master_anvil_command }}"

# Custom anvil process configuration
postfix_master_anvil__custom: {}

# Final anvil process configuration
postfix_master_anvil: "{{
  postfix_master_anvil__default.update(postfix_master_anvil__custom) }}{{
  postfix_master_anvil__default }}"

# Default values of the bounce process configuration
postfix_master_bounce_type: unix
postfix_master_bounce_chroot: no
postfix_master_bounce_maxproc: 0
postfix_master_bounce_command: bounce

# Default bounce process configuration
postfix_master_bounce__default:
  type: "{{ postfix_master_bounce_type }}"
  chroot: "{{ postfix_master_bounce_chroot }}"
  maxproc: "{{ postfix_master_bounce_maxproc }}"
  command: "{{ postfix_master_bounce_command }}"

# Custom bounce process configuration
postfix_master_bounce__custom: {}

# Final bounce process configuration
postfix_master_bounce: "{{
  postfix_master_bounce__default.update(postfix_master_bounce__custom) }}{{
  postfix_master_bounce__default }}"

# Default values of the cleanup process configuration
postfix_master_cleanup_type: unix
postfix_master_cleanup_private: no
postfix_master_cleanup_chroot: no
postfix_master_cleanup_maxproc: 0
postfix_master_cleanup_command: cleanup

# Default cleanup process configuration
postfix_master_cleanup__default:
  type: "{{ postfix_master_cleanup_type }}"
  private: "{{ postfix_master_cleanup_private }}"
  chroot: "{{ postfix_master_cleanup_chroot }}"
  maxproc: "{{ postfix_master_cleanup_maxproc }}"
  command: "{{ postfix_master_cleanup_command }}"

# Custom cleanup process configuration
postfix_master_cleanup__custom: {}

# Final cleanup process configuration
postfix_master_cleanup: "{{
  postfix_master_cleanup__default.update(postfix_master_cleanup__custom) }}{{
  postfix_master_cleanup__default }}"

# Default values of the defer process configuration
postfix_master_defer_type: unix
postfix_master_defer_chroot: no
postfix_master_defer_maxproc: 0
postfix_master_defer_command: bounce

# Default defer process configuration
postfix_master_defer__default:
  type: "{{ postfix_master_defer_type }}"
  chroot: "{{ postfix_master_defer_chroot }}"
  maxproc: "{{ postfix_master_defer_maxproc }}"
  command: "{{ postfix_master_defer_command }}"

# Custom defer process configuration
postfix_master_defer__custom: {}

# Final defer process configuration
postfix_master_defer: "{{
  postfix_master_defer__default.update(postfix_master_defer__custom) }}{{
  postfix_master_defer__default }}"

# Default values of the discard process configuration
postfix_master_discard_type: unix
postfix_master_discard_chroot: no
postfix_master_discard_command: discard

# Default discard process configuration
postfix_master_discard__default:
  type: "{{ postfix_master_discard_type }}"
  chroot: "{{ postfix_master_discard_chroot }}"
  command: "{{ postfix_master_discard_command }}"

# Custom discard process configuration
postfix_master_discard__custom: {}

# Final discard process configuration
postfix_master_discard: "{{
  postfix_master_discard__default.update(postfix_master_discard__custom) }}{{
  postfix_master_discard__default }}"

# Default values of the error process configuration
postfix_master_error_type: unix
postfix_master_error_chroot: no
postfix_master_error_command: error

# Default error process configuration
postfix_master_error__default:
  type: "{{ postfix_master_error_type }}"
  chroot: "{{ postfix_master_error_chroot }}"
  command: "{{ postfix_master_error_command }}"

# Custom error process configuration
postfix_master_error__custom: {}

# Final error process configuration
postfix_master_error: "{{
  postfix_master_error__default.update(postfix_master_error__custom) }}{{
  postfix_master_error__default }}"

# Default values of the flush process configuration
postfix_master_flush_type: unix
postfix_master_flush_private: no
postfix_master_flush_chroot: no
postfix_master_flush_wakeup: 1000?
postfix_master_flush_maxproc: 0
postfix_master_flush_command: flush

# Default flush process configuration
postfix_master_flush__default:
  type: "{{ postfix_master_flush_type }}"
  private: "{{ postfix_master_flush_private }}"
  chroot: "{{ postfix_master_flush_chroot }}"
  wakeup: "{{ postfix_master_flush_wakeup }}"
  maxproc: "{{ postfix_master_flush_maxproc }}"
  command: "{{ postfix_master_flush_command }}"

# Custom flush process configuration
postfix_master_flush__custom: {}

# Final flush process configuration
postfix_master_flush: "{{
  postfix_master_flush__default.update(postfix_master_flush__custom) }}{{
  postfix_master_flush__default }}"

# Default values of the lmtp process configuration
postfix_master_lmtp_type: unix
postfix_master_lmtp_chroot: no
postfix_master_lmtp_command: lmtp

# Default lmtp process configuration
postfix_master_lmtp__default:
  type: "{{ postfix_master_lmtp_type }}"
  chroot: "{{ postfix_master_lmtp_chroot }}"
  command: "{{ postfix_master_lmtp_command }}"

# Custom lmtp process configuration
postfix_master_lmtp__custom: {}

# Final lmtp process configuration
postfix_master_lmtp: "{{
  postfix_master_lmtp__default.update(postfix_master_lmtp__custom) }}{{
  postfix_master_lmtp__default }}"

# Default values of the local process configuration
postfix_master_local_type: unix
postfix_master_local_unpriv: no
postfix_master_local_chroot: no
postfix_master_local_command: local

# Default local process configuration
postfix_master_local__default:
  type: "{{ postfix_master_local_type }}"
  unpriv: "{{ postfix_master_local_unpriv }}"
  chroot: "{{ postfix_master_local_chroot }}"
  command: "{{ postfix_master_local_command }}"

# Custom local process configuration
postfix_master_local__custom: {}

# Final local process configuration
postfix_master_local: "{{
  postfix_master_local__default.update(postfix_master_local__custom) }}{{
  postfix_master_local__default }}"

# Default values of the pickup process configuration
postfix_master_pickup_type: fifo
postfix_master_pickup_private: no
postfix_master_pickup_chroot: no
postfix_master_pickup_wakeup: 60
postfix_master_pickup_maxproc: 1
postfix_master_pickup_command: pickup

# Default pickup process configuration
postfix_master_pickup__default:
  type: "{{ postfix_master_pickup_type }}"
  private: "{{ postfix_master_pickup_private }}"
  chroot: "{{ postfix_master_pickup_chroot }}"
  wakeup: "{{ postfix_master_pickup_wakeup }}"
  maxproc: "{{ postfix_master_pickup_maxproc }}"
  command: "{{ postfix_master_pickup_command }}"

# Custom pickup process configuration
postfix_master_pickup__custom: {}

# Final pickup process configuration
postfix_master_pickup: "{{
  postfix_master_pickup__default.update(postfix_master_pickup__custom) }}{{
  postfix_master_pickup__default }}"

# Default values of the proxymap process configuration
postfix_master_proxymap_type: unix
postfix_master_proxymap_chroot: no
postfix_master_proxymap_command: proxymap

# Default proxymap process configuration
postfix_master_proxymap__default:
  type: "{{ postfix_master_proxymap_type }}"
  chroot: "{{ postfix_master_proxymap_chroot }}"
  command: "{{ postfix_master_proxymap_command }}"

# Custom proxymap process configuration
postfix_master_proxymap__custom: {}

# Final proxymap process configuration
postfix_master_proxymap: "{{
  postfix_master_proxymap__default.update(postfix_master_proxymap__custom) }}{{
  postfix_master_proxymap__default }}"

# Default values of the proxywrite process configuration
postfix_master_proxywrite_type: unix
postfix_master_proxywrite_chroot: no
postfix_master_proxywrite_maxproc: 1
postfix_master_proxywrite_command: proxymap

# Default proxywrite process configuration
postfix_master_proxywrite__default:
  type: "{{ postfix_master_proxywrite_type }}"
  chroot: "{{ postfix_master_proxywrite_chroot }}"
  maxproc: "{{ postfix_master_proxywrite_maxproc }}"
  command: "{{ postfix_master_proxywrite_command }}"

# Custom proxywrite process configuration
postfix_master_proxywrite__custom: {}

# Final proxywrite process configuration
postfix_master_proxywrite: "{{
  postfix_master_proxywrite__default.update(postfix_master_proxywrite__custom) }}{{
  postfix_master_proxywrite__default }}"

# Default values of the qmgr process configuration
postfix_master_qmgr_type: fifo
postfix_master_qmgr_private: no
postfix_master_qmgr_chroot: no
postfix_master_qmgr_wakeup: 300
postfix_master_qmgr_maxproc: 1
postfix_master_qmgr_command: qmgr

# Default qmgr process configuration
postfix_master_qmgr__default:
  type: "{{ postfix_master_qmgr_type }}"
  private: "{{ postfix_master_qmgr_private }}"
  chroot: "{{ postfix_master_qmgr_chroot }}"
  wakeup: "{{ postfix_master_qmgr_wakeup }}"
  maxproc: "{{ postfix_master_qmgr_maxproc }}"
  command: "{{ postfix_master_qmgr_command }}"

# Custom qmgr process configuration
postfix_master_qmgr__custom: {}

# Final qmgr process configuration
postfix_master_qmgr: "{{
  postfix_master_qmgr__default.update(postfix_master_qmgr__custom) }}{{
  postfix_master_qmgr__default }}"

# Default values of the relay process configuration
postfix_master_relay_type: unix
postfix_master_relay_chroot: no
postfix_master_relay_command: smtp -o smtp_fallback_relay=

# Default relay process configuration
postfix_master_relay__default:
  type: "{{ postfix_master_relay_type }}"
  chroot: "{{ postfix_master_relay_chroot }}"
  command: "{{ postfix_master_relay_command }}"

# Custom relay process configuration
postfix_master_relay__custom: {}

# Final relay process configuration
postfix_master_relay: "{{
  postfix_master_relay__default.update(postfix_master_relay__custom) }}{{
  postfix_master_relay__default }}"

# Default values of the retry process configuration
postfix_master_retry_type: unix
postfix_master_retry_chroot: no
postfix_master_retry_command: error

# Default retry process configuration
postfix_master_retry__default:
  type: "{{ postfix_master_retry_type }}"
  chroot: "{{ postfix_master_retry_chroot }}"
  command: "{{ postfix_master_retry_command }}"

# Custom retry process configuration
postfix_master_retry__custom: {}

# Final retry process configuration
postfix_master_retry: "{{
  postfix_master_retry__default.update(postfix_master_retry__custom) }}{{
  postfix_master_retry__default }}"

# Default values of the rewrite process configuration
postfix_master_rewrite_type: unix
postfix_master_rewrite_chroot: no
postfix_master_rewrite_command: trivial-rewrite

# Default rewrite process configuration
postfix_master_rewrite__default:
  type: "{{ postfix_master_rewrite_type }}"
  chroot: "{{ postfix_master_rewrite_chroot }}"
  command: "{{ postfix_master_rewrite_command }}"

# Custom rewrite process configuration
postfix_master_rewrite__custom: {}

# Final rewrite process configuration
postfix_master_rewrite: "{{
  postfix_master_rewrite__default.update(postfix_master_rewrite__custom) }}{{
  postfix_master_rewrite__default }}"

# Default values of the scache process configuration
postfix_master_scache_type: unix
postfix_master_scache_chroot: no
postfix_master_scache_maxproc: 1
postfix_master_scache_command: scache

# Default scache process configuration
postfix_master_scache__default:
  type: "{{ postfix_master_scache_type }}"
  chroot: "{{ postfix_master_scache_chroot }}"
  maxproc: "{{ postfix_master_scache_maxproc }}"
  command: "{{ postfix_master_scache_command }}"

# Custom scache process configuration
postfix_master_scache__custom: {}

# Final scache process configuration
postfix_master_scache: "{{
  postfix_master_scache__default.update(postfix_master_scache__custom) }}{{
  postfix_master_scache__default }}"

# Default values of the showq process configuration
postfix_master_showq_type: unix
postfix_master_showq_private: no
postfix_master_showq_chroot: no
postfix_master_showq_command: showq

# Default showq process configuration
postfix_master_showq__default:
  type: "{{ postfix_master_showq_type }}"
  private: "{{ postfix_master_showq_private }}"
  chroot: "{{ postfix_master_showq_chroot }}"
  command: "{{ postfix_master_showq_command }}"

# Custom showq process configuration
postfix_master_showq__custom: {}

# Final showq process configuration
postfix_master_showq: "{{
  postfix_master_showq__default.update(postfix_master_showq__custom) }}{{
  postfix_master_showq__default }}"

# Default values of the smtp process configuration
postfix_master_smtp_type: inet
postfix_master_smtp_private: no
postfix_master_smtp_chroot: no
postfix_master_smtp_command: smtpd

# Default smtp process configuration
postfix_master_smtp__default:
  type: "{{ postfix_master_smtp_type }}"
  private: "{{ postfix_master_smtp_private }}"
  chroot: "{{ postfix_master_smtp_chroot }}"
  command: "{{ postfix_master_smtp_command }}"

# Custom smtp process configuration
postfix_master_smtp__custom: {}

# Final smtp process configuration
postfix_master_smtp: "{{
  postfix_master_smtp__default.update(postfix_master_smtp__custom) }}{{
  postfix_master_smtp__default }}"

# Default values of the smtp process configuration
postfix_master_smtpx_type: unix
postfix_master_smtpx_chroot: no
postfix_master_smtpx_command: smtp

# Default smtp process configuration
postfix_master_smtpx__default:
  type: "{{ postfix_master_smtpx_type }}"
  chroot: "{{ postfix_master_smtpx_chroot }}"
  command: "{{ postfix_master_smtpx_command }}"

# Custom smtp process configuration
postfix_master_smtpx__custom: {}

# Final smtp process configuration
postfix_master_smtpx: "{{
  postfix_master_smtpx__default.update(postfix_master_smtpx__custom) }}{{
  postfix_master_smtpx__default }}"

# Default values of the tlsmgr process configuration
postfix_master_tlsmgr_type: unix
postfix_master_tlsmgr_chroot: no
postfix_master_tlsmgr_wakeup: 1000?
postfix_master_tlsmgr_maxproc: 1
postfix_master_tlsmgr_command: tlsmgr

# Default tlsmgr process configuration
postfix_master_tlsmgr__default:
  type: "{{ postfix_master_tlsmgr_type }}"
  chroot: "{{ postfix_master_tlsmgr_chroot }}"
  wakeup: "{{ postfix_master_tlsmgr_wakeup }}"
  maxproc: "{{ postfix_master_tlsmgr_maxproc }}"
  command: "{{ postfix_master_tlsmgr_command }}"

# Custom tlsmgr process configuration
postfix_master_tlsmgr__custom: {}

# Final tlsmgr process configuration
postfix_master_tlsmgr: "{{
  postfix_master_tlsmgr__default.update(postfix_master_tlsmgr__custom) }}{{
  postfix_master_tlsmgr__default }}"

# Default values of the trace process configuration
postfix_master_trace_type: unix
postfix_master_trace_chroot: no
postfix_master_trace_maxproc: 0
postfix_master_trace_command: bounce

# Default trace process configuration
postfix_master_trace__default:
  type: "{{ postfix_master_trace_type }}"
  chroot: "{{ postfix_master_trace_chroot }}"
  maxproc: "{{ postfix_master_trace_maxproc }}"
  command: "{{ postfix_master_trace_command }}"

# Custom trace process configuration
postfix_master_trace__custom: {}

# Final trace process configuration
postfix_master_trace: "{{
  postfix_master_trace__default.update(postfix_master_trace__custom) }}{{
  postfix_master_trace__default }}"

# Default values of the verify process configuration
postfix_master_verify_type: unix
postfix_master_verify_chroot: no
postfix_master_verify_maxproc: 1
postfix_master_verify_command: verify

# Default verify process configuration
postfix_master_verify__default:
  type: "{{ postfix_master_verify_type }}"
  chroot: "{{ postfix_master_verify_chroot }}"
  maxproc: "{{ postfix_master_verify_maxproc }}"
  command: "{{ postfix_master_verify_command }}"

# Custom verify process configuration
postfix_master_verify__custom: {}

# Final verify process configuration
postfix_master_verify: "{{
  postfix_master_verify__default.update(postfix_master_verify__custom) }}{{
  postfix_master_verify__default }}"

# Default values of the virtual process configuration
postfix_master_virtual_type: unix
postfix_master_virtual_unpriv: no
postfix_master_virtual_chroot: no
postfix_master_virtual_command: virtual

# Default virtual process configuration
postfix_master_virtual__default:
  type: "{{ postfix_master_virtual_type }}"
  unpriv: "{{ postfix_master_virtual_unpriv }}"
  chroot: "{{ postfix_master_virtual_chroot }}"
  command: "{{ postfix_master_virtual_command }}"

# Custom virtual process configuration
postfix_master_virtual__custom: {}

# Final virtual process configuration
postfix_master_virtual: "{{
  postfix_master_virtual__default.update(postfix_master_virtual__custom) }}{{
  postfix_master_virtual__default }}"

# Default master configuration
postfix_master__default:
  - anvil: "{{ postfix_master_anvil }}"
  - bounce: "{{ postfix_master_bounce }}"
  - cleanup: "{{ postfix_master_cleanup }}"
  - defer: "{{ postfix_master_defer }}"
  - discard: "{{ postfix_master_discard }}"
  - error: "{{ postfix_master_error }}"
  - flush: "{{ postfix_master_flush }}"
  - lmtp: "{{ postfix_master_lmtp }}"
  - local: "{{ postfix_master_local }}"
  - pickup: "{{ postfix_master_pickup }}"
  - proxymap: "{{ postfix_master_proxymap }}"
  - proxywrite: "{{ postfix_master_proxywrite }}"
  - qmgr: "{{ postfix_master_qmgr }}"
  - relay: "{{ postfix_master_relay }}"
  - retry: "{{ postfix_master_retry }}"
  - rewrite: "{{ postfix_master_rewrite }}"
  - scache: "{{ postfix_master_scache }}"
  - showq: "{{ postfix_master_showq }}"
  - smtp: "{{ postfix_master_smtp }}"
  - smtp: "{{ postfix_master_smtpx }}"
  - tlsmgr: "{{ postfix_master_tlsmgr }}"
  - trace: "{{ postfix_master_trace }}"
  - verify: "{{ postfix_master_verify }}"
  - virtual: "{{ postfix_master_virtual }}"

# Custom master configuration
postfix_master__custom: []

# Final master configuration
postfix_master: "{{
  postfix_master__default +
  postfix_master__custom }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)
- [`stunnel`](https://github.com/jtyr/ansible-stunnel) (optional)


License
-------

MIT


Author
------

Jiri Tyr
