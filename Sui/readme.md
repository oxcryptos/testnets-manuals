# SUI installation commands


### PREPARATION


		
	sudo apt update && sudo apt upgrade -y
_
		
	sudo apt install wget jq git libclang-dev cmake -y
_
		
	apt install pkg-config libssl-dev build-essential -y
_
		
	curl https://sh.rustup.rs -sSf | sh
_
	
	source $HOME/.cargo/env
_
		
	rustc --version

### INSTALLATION
		
	mkdir -p $HOME/.sui
_
		
	git clone https://github.com/MystenLabs/sui

_
		
	cd sui

_
		
	git remote add upstream https://github.com/MystenLabs/sui

_
		
	git fetch upstream

_
		
	git checkout --track upstream/devnet

_
		
	cargo build --release

_
		
	mv $HOME/sui/target/release/{sui,sui-node,sui-faucet} /usr/bin/

_
		
	cd $HOME

_

	wget -qO $HOME/.sui/genesis.blob https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob

_

	cp $HOME/sui/crates/sui-config/data/fullnode-template.yaml \
	$HOME/.sui/fullnode.yaml
		
_

	sed -i -e "s%db-path:.*%db-path: \"$HOME/.sui/db\"%; "\
	"s%metrics-address:.*%metrics-address: \"0.0.0.0:9184\"%; "\
	"s%json-rpc-address:.*%json-rpc-address: \"0.0.0.0:9000\"%; "\
	"s%genesis-file-location:.*%genesis-file-location: \"$HOME/.sui/genesis.blob\"%; " $HOME/.sui/fullnode.yaml

_

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

_
	
	sudo systemctl daemon-reload
	sudo systemctl enable suid 
	sudo systemctl restart suid

_
		
	. <(wget -qO- https://raw.githubusercontent.com/SecorD0/utils/main/miscellaneous/insert_variable.sh) -n sui_log -v "sudo journalctl -fn 100 -u suid" -a

_
		
	wget -qO- -t 1 -T 5 --header 'Content-Type: application/json' --post-data '{ "jsonrpc":"2.0", "id":1, "method":"sui_getRecentTransactions", "params":[5] }' "http://127.0.0.1:9000/" | jq

### WALLET
