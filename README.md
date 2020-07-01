rwreceiver
=========

A role for configuring and managing the rwreceiver service.  rwreceiver is a daemon which accepts files transferred from one or more rwsender processes. The received files are stored in a destination directory. See rwreceiver [documentation](https://tools.netsa.cert.org/silk/rwreceiver.html) for more information.

Requirements
------------

If using TLS for connections, matching certs need to be generated and uploaded to both sender and receiver.

Role Variables
--------------

Available variables are listed below, along with default values (see [defaults/main.yml](defaults/main.yml)):

	silk_packing_tools_loc: "/usr/local/sbin"

The folder location of the silk packing tools.

	silk_tls_support: False

Whether to use TLS for connections.

	rwreceiver_myname: "rwreceiver"

The name of the rwreceiver process.  It is possible to run multiple copies of rwreceiver on a single box by naming them differently.

	rwreceiver_conf_template: "rwreceiver.conf.j2"
	rwreceiver_conf_file_loc: "/usr/local/etc"
	rwreceiver_conf_file_path: "{{ rwreceiver_conf_file_loc }}/{{ rwreceiver_myname }}.conf"
	rwreceiver_init_template: "rwreceiver.j2"
	rwreceiver_init_file_path: "/etc/init.d/{{ rwreceiver_myname }}" 

Template sources to use and their destinations.

| Variable  | Explanation |
| ------------- | ------------- |
| rwreceiver_statedirectory: "/usr/local/var/lib/rwreceiver" | Directory for where rwreceiver keeps its state |
| rwreceiver_create_directories: "no" | If set to "yes", the defined directories will be created automatically if they do not already exist |
| rwreceiver_bin_dir: "{{ silk_packing_tools_loc }}" | Full path of the directory containing the "rwreceiver" program |
| rwreceiver_destination_dir: "{{ rwreceiver_statedirectory }}/destination" | The full path of the directory where received files will be placed |
| rwreceiver_mode: "client" | The mode in which the receiver will run.  Valid values are "server" and "client". |
| rwreceiver_id: "receiver-1" | The name of this receiver instance |
| rwreceiver_port: "" | The PORT or HOST:PORT pair upon which the server listens for incoming connections.  This is only required when running in server mode.  If HOST is not provided, the server listens on any address. HOST may be a name or an IP address. If HOST is an IPv6 address, enclose it in square brackets, and enclose the entire value in single quotes to prevent interpretation by the shell. |
| rwreceiver_post_command: "" | Command to run on each file once it has been received.  In the command, "%s" will be replaced with the full path to the received file, and "%I" will be replaced with the identifier of the rwsender that sent the file. E.g.: POST_COMMAND='echo received file %s from rwsender %I' |
| rwreceiver_freespace_min: "0" | The amount of space (in bytes) rwreceiver will attempt to leave free in the file system containing $DESTINATION_DIR.  This variable may be set as an ordinary integer, or as a real number followed by a suffix K, M, G, or T. |
| rwreceiver_space_max_percent: "100" | The maximum percentage of space of the file system containing $DESTINATION_DIR that rwreceiver will be willing to use. |
| rwreceiver_sender_servers: "" | If the receiver is in client mode, then SENDER_SERVERS must be specified.  The lines should have the following format: `<identifier> <host>:<port>`. These variables are multi-line values, where each line has the value for one rwsender. |
| rwreceiver_sender_clients: "" | If the receiver is in server mode, then SENDER_CLIENTS must be specified.  The lines should have the following format: `<identifier>`. These variables are multi-line values, where each line has the value for one rwsender. |
| rwreceiver_duplicate_dirs: "" | The receiver can copy incoming files to multiple destination directories.  Note that if copying to one of the following directories fails, an error is logged but the file is treated as having been successfully transferred. This variable contains a multi-line value, where each line lists a duplicate destination directory.  The format for the line is: `<duplicate-dir>`. Note that each line must be a complete directory path |
| rwreceiver_duplicate_copies: "link" | When copying files to a duplicate destination directory, by default rwreceiver creates the files as hard links (if possible) to each other and to files in the destination directory.  However, if some process modifies-in-place a copy in one location that will affect all files.  To force rwreceiver to create a complete copy, change DUPLICATE_COPIES from "link" to "copy". |
| rwreceiver_log_type: "syslog" | The type of logging to use.  Valid values are "legacy" and "syslog". |
| rwreceiver_log_level: "info" | The lowest level of logging to actually log.  Valid values are: emerg, alert, crit, err, warning, notice, info, debug |
| rwreceiver_log_dir: "{{ rwreceiver_statedirectory }}/log" | The full path of the directory where the log files will be written when LOG_TYPE is "legacy". |
| rwreceiver_pid_dir: "{{ rwreceiver_log_dir }}" | The full path of the directory where the PID file will be written |
| rwreceiver_user: "root" | The user this program runs as; root permission is required only when rwreceiver listens on a privileged port. |
| rwreceiver_extra_options: "" | Extra options to pass to rwreceiver |

When using the optional GnuTLS support by setting `silk_tls_support: True`, the full path the CA file (TLS_CA) must be specified, as well as **EITHER** the full path to the program-specific PKCS#12 file (TLS_PKCS12) *OR* the full paths to the program-specific certificate (TLS_CERT) and key (TLS_KEY) files. If the PKCS#12 file is password protected, you must set the RWRECEIVER_TLS_PASSWORD environment variable to the password prior to starting rwreceiver.  If RWRECEIVER_TLS_PASSWORD is not set, it is treated as a null password; set it to the empty string to allow for an empty password.


| TLS Variable  | Explanation |
| ------------- | ------------- |
| rwreceiver_tls_ca: "" | The full path to the root CA file, PEM encoded |
| rwreceiver_tls_pkcs: "" | The full path to the program specific PKCS#12 file, DER encoded |
| rwreceiver_tls_key: "" | The full path to the program specific key file, PEM encoded |
| rwreceiver_tls_cert: "" | The full path to the program specific certificate file, PEM encoded |
| rwreceiver_tls_crl: "" | The full path to the Certificate Revocation List, PEM encoded. Optional. |
| rwreceiver_tls_priority: "" | The preference order (priority) for ciphers, key exchange methods, message authentication codes, and compression methods.  Optional. The default is "NORMAL".  Available arguments depend on your version of GnuTLS. |
| rwreceiver_tls_security: "" | The security level to use when generating Diffie-Hellman parameters. One of low, medium, high, or ultra.  Optional.  The default is "medium". |
| rwreceiver_tls_debug_level: "" | The debugging level used internally by the GnuTLS library, a number between 0 (no debugging) and 99 inclusive.  Optional. |



Dependencies
------------

- cmusei.silk

Example Playbook
----------------

    - hosts: server
      become: true
      vars:
        data_root_dir: "/data"
        # rwreceiver settings
        silk_tls_support: True
        rwreceiver_statedirectory: "{{ data_root_dir }}/rwreceiver"
        rwreceiver_destination_dir: "{{ rwreceiver_statedirectory }}/incoming"
        rwreceiver_create_directories: "yes"
        rwreceiver_mode: "server"
        rwreceiver_port: "3737"
        rwreceiver_sender_clients: |
            sender1
            sender2
        # rwreceiver tls settings
        tls_ca: "testcert.pem"
        tls_key: "client-key.pem"
        tls_cert: "client-cert.pem"
        rwreceiver_tls_ca: "/etc/pki/tls/{{ tls_ca }}"
        rwreceiver_tls_key: "/etc/pki/tls/private/{{ tls_key }}"
        rwreceiver_tls_cert: "/etc/pki/tls/{{ tls_cert }}"
        rwreceiver_pid_dir: "/var/run"
      pre_tasks:
        - name: Copy ssl certs
          copy:
            src: "{{ item.f }}"
            dest: "{{ item.d }}"
            mode: "{{ item.m }}"
            owner: "root"
            group: "root"
          with_items:
            - f: "{{ tls_ca }}"
              d: "{{ rwreceiver_tls_ca }}"
              m: '0644'
            - f: "{{ tls_key }}"
              d: "{{ rwreceiver_tls_key }}"
              m: '0600'
            - f: "{{ tls_cert}}"
              d: "{{ rwreceiver_tls_cert }}"
              m: '0644'
      roles:
        - role: cmusei.rwreceiver
          tags: [ 'rwreceiver' ]

License
-------

Copyright 2020 Carnegie Mellon University.
NO WARRANTY. THIS CARNEGIE MELLON UNIVERSITY AND SOFTWARE ENGINEERING INSTITUTE MATERIAL IS FURNISHED ON AN "AS-IS" BASIS. CARNEGIE MELLON UNIVERSITY MAKES NO WARRANTIES OF ANY KIND, EITHER EXPRESSED OR IMPLIED, AS TO ANY MATTER INCLUDING, BUT NOT LIMITED TO, WARRANTY OF FITNESS FOR PURPOSE OR MERCHANTABILITY, EXCLUSIVITY, OR RESULTS OBTAINED FROM USE OF THE MATERIAL. CARNEGIE MELLON UNIVERSITY DOES NOT MAKE ANY WARRANTY OF ANY KIND WITH RESPECT TO FREEDOM FROM PATENT, TRADEMARK, OR COPYRIGHT INFRINGEMENT.
Released under a MIT (SEI)-style license, please see license.txt or contact permission@sei.cmu.edu for full terms.
[DISTRIBUTION STATEMENT A] This material has been approved for public release and unlimited distribution.  Please see Copyright notice for non-US Government use and distribution.
CERT® is registered in the U.S. Patent and Trademark Office by Carnegie Mellon University.
This Software includes and/or makes use of the following Third-Party Software subject to its own license:
1. ansible (https://github.com/ansible/ansible/tree/devel/licenses) Copyright 2019 Red Hat, Inc.
2. molecule (https://github.com/ansible-community/molecule/blob/master/LICENSE) Copyright 2018 Red Hat, Inc.
3. testinfra (https://github.com/philpep/testinfra/blob/master/LICENSE) Copyright 2020 Philippe Pepiot.

DM20-0505


Author Information
------------------

This role was created in 2019 by [Matt Heckathorn](https://resources.sei.cmu.edu/library/author.cfm?authorID=2403).
