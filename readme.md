## Tezos Reward Distributor : Run & Forget

DISCLAIMER : TEZOS REWARD DISTRIBUTOR IS PROVIDED AS IS. IT IS UNDER CONSTANT DEVELOPMENT. EVENT THOUGH IT IS WELL TESTED, PLEASE USE WITH CARE. ALWAYS MAKE A PRE-RUN IN DRY MODE BEFORE MAKING ACTUAL PAYMENTS. IF YOU WANT TO RUN IN SERVICE MODE DO IT AFTER YOU ARE CONFIDENT WITH THE APPLICATION AND IN SERVICE MODE DO NOT UPDATE OFTEN.


### Tezos Reward Distributor

A software for distributing baking rewards with delegators. This is not a script but service which can run in the background all the time. It can track cycles and make payments. It does not have to be used as a service, It can also be used interactively. 

Design principals are: 

- Automatic Execution with no manual involvement: Run and forget
- Simplicity: Simple and intuitional parameter configuration. Default values are ready for the most common use case. 
- Covering different use cases: supporters, special rates, future payments, security deposit owned by multiple parties, fee sharing among founders. Choose which cycle to pay and when to stop the application.
- Traceability: CSV payment reports with calculation details. Logs for traking application lifecycle.
- Testability: Dry for seeing results witout making any modification. Support for development networks e.g. zeronet, alphanet.
- Safety: Payment logs for avoiding multiple payments by mistake. Lock file for avoiding multiple instances running at the same time. Shutdown handlers for avoiding application shutdowns in the middle of a sensitive operation. 

Features:
- Reward calculations based on tzscan API.
- Batch Payments
- Email notifications
- Re-attempt failed payments
- Minimal configuration needs, while having many configuration options
- Written in Python. Easy to modify to suit custom needs


### Requirements and Setup:

Python 3 is required. Download the application repository using git clone:

```
git clone https://github.com/habanoz/tezos-reward-distributor
```

To install required modules, use pip with requirements.txt provided.

```
cd tezos-reward-distributor
pip3 install -r requirements.txt
```

Regulary check and upgrade to the latest available version:

```
git pull
```

### How to Run:

For a list of parameters, run:

```
python3 src/main.py --help
```

The most common use case is to run in mainnet and start to make payments from last released rewards or continue making payments from the cycle last payment is done. 

```
python3 src/main.py
```

Make payments for a single cycle:

```
python3 src/main.py -C 42 -M 3 
```

Make pending payments and stop:

```
python3 src/main.py -M 2
```

Make pending payments beginning from a cycle and stop:

```
python3 src/main.py -C 30 -M 2
```

Run in dry-run mode in zeronet, make payments from cycle 30 and exit:

```
python3 src/main.py -D -N ZERONET -C 30 -M 3
```

### Baking Configuration:

Each baker has its own policy. A payment system should be flexable enough to cover needs of bakers. The applcation uses a yaml file for loading baker specific configurations. 

By default configuration files are kept under ~/pymnt/cfg directory. Configuration directory can be changed with "-f" configuration option. Name of a configuration file should be the baker's address (e.g. tz1boot1pK9h2BVGXdyvfQSv8kd1LQM6H889.yaml).  

For example configuration, please see tz1boot1pK9h2BVGXdyvfQSv8kd1LQM6H889.yaml file in repository home directory. You can start by renaming and copying the example file to configuration directory. 

Available configuration parameters are:
- baking_address : Address of the baker. It must be an implicit account (tz1). No alias is allowed.
- payment_address : This is the address where payments will done. A PKH or alias of implicit or originated account is accepted. An address can only be used for payments if it satifies following criteria:
  - Public key of the address must be revealed. See tezos command line interface on how to run reveal command on tezos client. If an address is registered as delegate, there is no need to run reveal command.
  - Secret key of the address must be known. If the payment address is an implicit address (tz), its secret key must be imported. If payment address is an originated address (KT), secret key of the manager address must be imported.
  - If secret key is encrypted, tezos-signer must be used to sign payments. 
 - owners_map : A dictionary of PKH and ratio ( decimal in range [0-1]) pairs. Each item in this dict represents PKH of each balance owner and his ratio of the amount he owned in the total baking balance. Implicit or originated addresses are accepted. It is important that sum of all ratios equals to 1. This map is optional if owner do not want to be paid for baking rewards, in this case service fee remains in baking balance.
 - founders_map : A dictionary of PKH and ratio (decimal in range [0-1]) pairs. Each item in this dict represents PKH of each founder and his ratio of the shares coming from service fees. Implicit or originated addresses are accepted. It is important that sum of all ratios equals to 1. This map is optional if founders do not want to be paid from service fees, in this case service fee remains in baking balance.
 - service_fee : A decimal in range [0-100]. This is evaluated as a percentage value. If this value is set to 5, 5% of baking rewards is kept as service fee.
 - supporters_set : A set of PKH values. Each PKH represents a supporter of the baker. Supporters are not charged with service fee. Founders and balance owners are natural supporters, they are not needed to be added. 
 - specials_map : A dictionary of PKH and fee (decimal in range [0-100] ) pairs. This dictionary can be used to set special service fee values for desired delegators.
 - pymnt_scale : This parameter is used to set scaling for payment calculations. A payment amount is rounded down according to this scale. For example if payment amount is 3.45678 and pymnt_scale is 3, actual payment amount will be 3.456. If you want to turn rounding off set this value to None.
 - prcnt_scale : This parameter is used to set scaling for percentage calculations. A percentage amount is rounded down according to this scale. For example if a delegator owns 4.123456 % of total rewards and prcnt_scale is 4, actual reward percentage will be 4.1234. If you want to turn rounding off set this value to None.
 - delegator_pays_xfer_fee : Default value is true. If set to false, transfer fee for each payment is paid by the delegate. Otherwise, transfer fee is deducted from the delegator reward.


### Linux Service

It is possible to add tezos-reward-distributer as a Linux service. It can run in the background. In order to set up the service with default configuration arguments, run the following command:

```
sudo python3 service_add.py
```

Note: If you do not want to use the default arguments, append any arguments you wish to change after service_add.py. They will be appended to main.py call.


It will create a service file and use it to enable the service. Once enabled use following commands to start/stop the service.

```
sudo systemctl start tezos-reward.service
sudo systemctl stop tezos-reward.service
```

In order to see service status:

```
systemctl status tezos-reward.service
```

In order to see logs:

```
journalctl --follow --unit=tezos-reward.service
```

### Email Setup

tezos-reward-distribute will create an email.ini file. Fill this file with your email configuration to send payment emails.

### Fee Setup

fee.ini file contains details about transaction fees. Currently the fee value specified under DEFAULT domain is used as fee amount. It is in mutez. Check the link below to see effect of fee value of 1274.

https://zeronet.tzscan.io/opCnDj8bpr5ACrbLSqy4BDCMsNiY8Y34bvnm2hj7MvcxaRiu5tu


### Contributions
Please refer to contributions guide on wiki pages.

https://github.com/habanoz/tezos-reward-distributor/wiki/How-to-Contribute

#### Terms:

- Reward: Coins rewarded by the network for the baking/endorsing operations.
- Payment: Coins paid to delegators after excluding service fee.
- freeze cycle: number of cycles rewards are kept frozen by the tezos network. Can be given negative values to let the application make future payments.
