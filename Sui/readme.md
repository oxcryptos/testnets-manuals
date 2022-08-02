# SUI installation commands


### PREPARATION
-
		sudo apt update && sudo apt upgrade -y
-
		sudo apt install wget jq git libclang-dev cmake -y
-
		apt install pkg-config libssl-dev build-essential -y
-
		curl https://sh.rustup.rs -sSf | sh
-
		source $HOME/.cargo/env
-
		rustc --version

### INSTALLATION
-
		mkdir -p $HOME/.sui
-
		git clone https://github.com/MystenLabs/sui

-
		cd sui

-
		git remote add upstream https://github.com/MystenLabs/sui

-
		git fetch upstream

-
		git checkout --track upstream/devnet

-
		cargo build --release

-
		mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/bin/

-
		cd $HOME

-
		wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob

-
		cp $HOME/sui/crates/sui-config/data/fullnode-template.yaml \
		$HOME/.sui/fullnode.yaml

-
		sed -i -e "s%db-path:.*%db-path: \"$HOME/.sui/db\"%; "\
		"s%metrics-address:.*%metrics-address: \"0.0.0.0:9184\"%; "\
		"s%json-rpc-address:.*%json-rpc-address: \"0.0.0.0:9000\"%; "\
		"s%genesis-file-location:.*%genesis-file-location: \"$HOME/.sui/genesis.blob\"%; " $HOME/.sui/fullnode.yaml

-
		printf "[Unit]
		Description=Sui node
		After=network-online.target

		[Service]
		User=$USER
		ExecStart=`which sui-node` --config-path $HOME/.sui/fullnode.yaml
		Restart=on-failure
		RestartSec=3
		LimitNOFILE=65535

		[Install]
		WantedBy=multi-user.target" > /etc/systemd/system/suid.service

-
		sudo systemctl daemon-reload
		sudo systemctl enable suid 
		sudo systemctl restart suid

-
		. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/miscellaneous/insert_variable.sh) -n sui_log -v "sudo journalctl -fn 100 -u suid" -a

-
		wget -qO- -t 1 -T 5 --header 'Content-Type: application/json' --post-data '{ "jsonrpc":"2.0", "id":1, "method":"sui_getRecentTransactions", "params":[5] }' "http://127.0.0.1:9000/" | jq

### WALLET
