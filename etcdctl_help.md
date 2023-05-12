 NAME:
        etcdctl - A simple command line client for etcd3.

USAGE:
        etcdctl [flags]

VERSION:
        3.5.7

API VERSION:
        3.5


COMMANDS:

        alarm disarm            Disarms all alarms        
        alarm list              Lists all alarms
        auth disable            Disables authentication
        auth enable             Enables authentication
        auth status             Returns authentication status
        check datascale         Check the memory usage of holding data for different workloads on a given server endpoint.
        check perf              Check the performance of the etcd cluster
        compaction              Compacts the event history in etcd
        defrag                  Defragments the storage of the etcd members with given endpoints
        del                     Removes the specified key or range of keys [key, range_end)
        elect                   Observes and participates in leader election
        endpoint hashkv         Prints the KV history hash for each endpoint in --endpoints
        endpoint health         Checks the healthiness of endpoints specified in `--endpoints` flag
        endpoint status         Prints out the status of endpoints specified in `--endpoints` flag
        get                     Gets the key or a range of keys
        help                    Help about any command
        lease grant             Creates leases
        lease keep-alive        Keeps leases alive (renew)
        lease list              List all active leases
        lease revoke            Revokes leases
        lease timetolive        Get lease information
        lock                    Acquires a named lock
        make-mirror             Makes a mirror at the destination etcd cluster
        member add              Adds a member into the cluster
        member list             Lists all members in the cluster
        member promote          Promotes a non-voting member in the cluster
        member remove           Removes a member from the cluster
        member update           Updates a member in the cluster
        move-leader             Transfers leadership to another etcd cluster member.
        put                     Puts the given key into the store
        role add                Adds a new role
        role delete             Deletes a role
        role get                Gets detailed information of a role
        role grant-permission   Grants a key to a role
        role list               Lists all roles
        role revoke-permission  Revokes a key from a role
        snapshot restore        Restores an etcd member snapshot to an etcd directory
        snapshot save           Stores an etcd node backend snapshot to a given file
        snapshot status         [deprecated] Gets backend snapshot status of a given file
        txn                     Txn processes all the requests in one transaction
        user add                Adds a new user
        user delete             Deletes a user
        user get                Gets detailed information of a user
        user grant-role         Grants a role to a user
        user list               Lists all users
        user passwd             Changes password of user
        user revoke-role        Revokes a role from a user
        version                 Prints the version of etcdctl
        watch                   Watches events stream on keys or prefixes

OPTIONS:

      --cacert=""                               verify certificates of TLS-enabled secure servers using this CA bundle
      --cert=""                                 identify secure client using this TLS certificate file
      --command-timeout=5s                      timeout for short running command (excluding dial timeout)
      --debug[=false]                           enable client-side debug logging
      --dial-timeout=2s                         dial timeout for client connections
      
  -d, 
      --discovery-srv=""                        domain name to query for SRV records describing cluster endpoints
      --discovery-srv-name=""                   service name to query when using DNS discovery
      --endpoints=[127.0.0.1:2379]              gRPC endpoints
      
  -h, 
      --help[=false]                            help for etcdctl
      --hex[=false]                             print byte strings as hex encoded strings
      --insecure-discovery[=true]               accept insecure SRV records describing cluster endpoints
      --insecure-skip-tls-verify[=false]        skip server certificate verification (CAUTION: this option should be enabled only for testing purposes)
      --insecure-transport[=true]               disable transport security for client connections
      --keepalive-time=2s                       keepalive time for client connections
      --keepalive-timeout=6s                    keepalive timeout for client connections
      --key=""                                  identify secure client using this TLS key file
      --password=""                             password for authentication (if this option is used, --user option shouldn't include password)
      --user=""                                 username[:password] for authentication (prompt if password is not supplied)
  -w, 
      --write-out="simple"                      set the output format (fields, json, protobuf, simple, table)
