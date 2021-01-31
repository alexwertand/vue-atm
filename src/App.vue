<template>
	<div id="app">
		<b-container>
			<b-row>
				<b-col class="mt-3 p-5 bg-light shadow-sm">
					<h1 class="display-4 text-center">ATM</h1>

					<hr class="my-4">

					<div class="d-flex flex-wrap">
						<div class="w-100 h2 text-center p-3">
							<span class="text-info">Total money: </span>

							<span class="text-primary">{{ totalMoney }}</span>
						</div>

						<hr class="w-100 my-2">

						<div
							v-for="(item, index) in atm"
							:key="index"
							class="w-50 h4 p-3">
							<p>
								<span class="mr-2 text-secondary">Banknote:</span>

								<span class="text-primary">{{ item.banknote }}</span>

							</p>
							
							<p>
								<span class="mr-2 text-secondary">Amount:</span>

								<span class="text-primary">{{ item.amount }}</span>
							</p>

							<p>
								<span class="mr-2 text-secondary">Total:</span>

								<span class="text-primary">{{ item.total }}</span>
							</p>

							<hr class="my-1">
						</div>
					</div>

					<div class="d-flex mt-1 p-3">
						<b-form-input
							v-model.number="sumToWithdraw"
							type="number"
							placeholder="Enter amount of money"
							class="w-75 mr-5">
						</b-form-input>

						<b-button
							@click="toWithdrawMoney"
							:disabled="isDisabledToWithdrawBtn"
							variant="success"
							class="w-25">Withdraw Money</b-button>
					</div>
				</b-col>
			</b-row>
		</b-container>
	</div>
</template>

<script>
    const cloneDeep = require('lodash.clonedeep');

	export default {
		name: 'App',
		data() {
			return {
				banknotes: [500, 200, 100, 50, 20, 10],
                atm: [],
                virtualAtm: [],
				totalMoney: 0,
                sumToWithdraw: null,
                isDisabledToWithdrawBtn: false,
                transaction: null,
			}
		},
		created() {
            this.init(20);
			this.countMoney();
            
            this.virtualAtm = cloneDeep(this.atm);

            this.transaction = this.initTransaction();
        },
		methods: {
			init(amount) {
				this.banknotes.forEach((banknote, index) => {
					this.$set(this.atm, index, { banknote, amount })
				});
			},
			countMoney() {
				this.totalMoney = 0;

				this.atm.forEach(item => {
                    let total = item.banknote * item.amount;
                    this.$set(item, 'total', total);
					this.totalMoney += total;
				});
			},
			toWithdrawMoney() {
                this.isDisabledToWithdrawBtn = true;

                if (this.sumToWithdraw > this.totalMoney) {
                    this.showMsg('Erorr occured!', 'ATM does not have enough money!');
                } else {
                    let virtualSumToWithdraw = this.sumToWithdraw;

                    this.virtualAtm.forEach(item => {
                        let remainder = Math.trunc(virtualSumToWithdraw / item.banknote);

                        if ( remainder >= 1 ) {

                            while ((item.amount && virtualSumToWithdraw > 0) && virtualSumToWithdraw >= item.banknote) {
                                item.amount--;

                                this.transaction.banknotes.add(item.banknote);

                                virtualSumToWithdraw -= item.banknote;

                                this.transaction.amount += item.banknote;
                            }
                        }
                    });

                    if (virtualSumToWithdraw > 0) {
                        this.showMsg('Erorr occured!', 'ATM does not have needed banknotes!');

                        this.virtualAtm = cloneDeep(this.atm);

                        this.transaction = this.initTransaction();
                    } else {
                        this.showMsg(
                            'Congratulations!',
                            `You cashed ${this.transaction.amount}! Banknotes: ${[...this.transaction.banknotes].join(', ')}`
                        );

                        this.atm = cloneDeep(this.virtualAtm);
                        this.countMoney();

                        this.transaction = this.initTransaction();
                    }
                }
            },
            initTransaction() {
                return {
                    amount: null,
                    banknotes: new Set
                }
            },
            showMsg(title, message) {
                this.$bvModal.msgBoxOk(message, {
                    title: title,
                    size: 'md',
                    buttonSize: 'md',
                    okVariant: 'success',
                    headerClass: 'p-3',
                    footerClass: 'p-3',
                    centered: true
                })
                .then(() => {
                    this.sumToWithdraw = null;
                    this.isDisabledToWithdrawBtn = false;
                })
                .catch(error => {
                    console.log(error);
                });
            }
		}
	}
</script>

<style>
</style>
