#!/bin/sh

set -eu

status () {
  echo "---> ${@}" >&2
}

if [ -z ${CLUSTER+x} ]; then
    echo "CLUSTER variable must be BOOT, INIT or cluster node list"
    exit 1
elif [ ${CLUSTER} = "BOOT" ]; then
    ln -sf /etc/mysql/my-init.cnf /etc/mysql/my.cnf
else
    ln -sf /etc/mysql/my-cluster.cnf /etc/mysql/my.cnf
fi

mv /usr/bin/xtrabackup /usr/bin/xtrabackup_51
ln -sf /usr/bin/xtrabackup_55 /usr/bin/xtrabackup

if [ -d /etc/ssl/mysql ]; then
   chown mysql:mysql /etc/ssl/mysql/*.pem
fi

# create link for less flexible scripts

ln -sf /var/lib/mysql/mysql.sock /var/run/mysqld/mysqld.sock

if [ -z ${CLUSTER+x} ]; then
    echo "CLUSTER variable must be BOOT, INIT or cluster node list"
    exit 1
elif [ ${CLUSTER} = "BOOT" ]; then
    if [ ! -e /var/lib/mysql/bootstrapped ]; then
        status "Bootstrapping MariaDB installation..."

        status "Initializing MariaDB root directory at /var/lib/mysql"
        mysql_install_db

        status "Setting MariaDB root password"
        /usr/bin/mariadb-setrootpassword
        touch /var/lib/mysql/bootstrapped
    else
        status "Starting from already-bootstrapped MariaDB installation"
    fi

    exec /usr/bin/mysqld_safe --log-error=/var/lib/mysql/mysql.error.log
elif [ ${CLUSTER} = "INIT" ]; then
    NODE=1 # force this
    # grab the ip address of eth0 if NODE_ADDR is not provided 
    NODE_ADDR=${NODE_ADDR:-$(ip -4 addr show eth0 | grep inet | sed -e 's/  */ /g' | cut -d ' ' -f 3 | cut -d '/' -f 1)}
    exec /usr/bin/mysqld_safe --wsrep_node_address="${NODE_ADDR}" \
        --wsrep_node_incoming_address="${NODE_ADDR}" \
        --wsrep_new_cluster --wsrep_cluster_address="gcomm://" \
        --log-error=/var/lib/mysql/mysql-$NODE.error.log
else
    if [ -z ${NODE+x} ]; then
        echo "Must define node number" >&2
        exit 1
    fi
    NODE_ADDR=${NODE_ADDR:-$(ip -4 addr show eth0 | grep inet | sed -e 's/  */ /g' | cut -d ' ' -f 3 | cut -d '/' -f 1)}
    CLUSTER="gcomm://$CLUSTER"
    exec /usr/bin/mysqld_safe --wsrep_node_address="${NODE_ADDR}" \
        --wsrep_node_incoming_address="${NODE_ADDR}" \
        --wsrep_cluster_address=$CLUSTER --log-error=/var/lib/mysql/mysql-$NODE.error.log
fi

