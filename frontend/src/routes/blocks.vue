<template>
    <!-- https://etherscan.io/blocks -->
    <div class=vue-blocks>
        <vue-bread v-bind:arr=breadcrumb title="Blocks"></vue-bread>

        <div class="container mt20">
            <div class="align-items-center info-and-pagination row">
                <div class=col>Showing Block (#{{ heightFrom }} to #{{ heightTo }}) out of {{ totalBlocks }} total blocks</div>
                <vue-pagination class=col-auto v-bind:current=currentPage v-bind:total=totalPage v-on:first=onFirst v-on:last=onLast v-on:next=onNext v-on:prev=onPrev v-on:to=onTo></vue-pagination>
            </div>
            <table class="mt20 table">
                <tr>
                    <th>Height</th>
                    <th class=text-right>Age</th>
                    <th>txn</th>
                    <th>Mined</th>
                    <th class=text-right>Gas Reward</th>
                    <th class=text-right>GasLimit</th>
                    <th class=text-right>Avg.GasPrice</th>
                </tr>
                <tr v-for="o in arr">
                    <td>
                        <router-link v-bind:to='fragApi + "/block/" + o.height'>{{ o.height }}</router-link>
                    </td>
                    <td class=time>
                        <div class=text-right>{{ timeConversion( Date.now() - o.timestamp) }} ago</div>
                        <div>{{ new Date(o.timestamp).toString() }} | {{ o.timestamp }}</div>
                    </td>
                    <td>
                        <router-link v-bind:to='fragApi + "/txs?block=" + o.height'>{{ o.txnCnt }}</router-link>
                    </td>
                    <td class=monospace>
                        <router-link v-bind:to='fragApi + "/address/" + o.miner.hash'>{{ o.miner.alias || o.miner.hash }}</router-link>
                    </td>
                    <td class=text-right>{{ toWei(o.gasReward) }}</td>
                    <td class=text-right>{{ numberAddComma(o.gasLimit) }}</td>
                    <td class=text-right>{{ toWei(o.avgGasPrice) }}</td>
                </tr>
            </table>
            <vue-pagination v-bind:current=currentPage right=1 v-bind:total=totalPage v-on:first=onFirst v-on:last=onLast v-on:next=onNext v-on:prev=onPrev v-on:to=onTo></vue-pagination>
        </div>
    </div>
</template>
<script>
    var api = require("@/assets/api"),
        utility = require("@/assets/utility");

    module.exports = {
        components: {
            "vue-bread": require("@/components/vue-bread").default,
            "vue-pagination": require("@/components/vue-pagination").default
        },
        data() {
            return {
                arr: [],
                breadcrumb: [
                    { text: "Home", to: "/" },
                    { text: "Blocks", to: "" }
                ],
                currentPage: 0,
                fragApi: this.$route.params.api ? "/" + this.$route.params.api : "",
                heightFrom: 0,
                heightTo: 0,
                totalBlocks: 0,
                totalPage: 0
            };
        },
        methods: {
            nthPage() {
                var p = this.$route.query.p || 1;

                if (p == this.currentPage)
                    console.log("nthPage - ????????????", p, "??????????????????, ??????????????????");
                else {
                    this.$root.showModalLoading = true;

                    api.getBlock({ p }, o => {

                        this.$root.showModalLoading = false;
                        this.arr = o.data;
                        this.currentPage = o.page;
                        this.totalPage = o.totalPage;
                        this.totalBlocks = o.totalCount;

                        if (this.arr.length) {
                            this.heightFrom = this.arr[0].height;
                            this.heightTo = this.arr[this.arr.length - 1].height;
                        } else {
                            this.heightFrom = 0;
                            this.heightTo = 0;
                        }
                    }, xhr => {
                        console.log(xhr);
                        this.$root.showModalLoading = false;
                        this.$router.replace((this.$route.params.api ? "/" + this.$route.params.api : "") + "/404!" + this.$route.fullPath);
                    });
                }
            },
            numberAddComma(n) {
                return utility.numberAddComma(n);
            },
            onFirst() {
                this.$router.push({
                    path: this.$route.path,
                    query: { p: 1 }
                });
            },
            onLast() {
                this.$router.push({
                    path: this.$route.path,
                    query: { p: this.totalPage }
                });
            },
            onNext() {
                this.$router.push({
                    path: this.$route.path,
                    query: { p: this.currentPage + 1 }
                });
            },
            onPrev() {
                this.$router.push({
                    path: this.$route.path,
                    query: { p: this.currentPage - 1 }
                });
            },
            onTo(n) {
                this.$router.push({
                    path: this.$route.path,
                    query: { p: n }
                });
            },
            timeConversion(ms) {
                return utility.timeConversion(ms);
            },
            toWei(n) {
                return utility.toWei(n);
            }
        },
        mounted() {
            this.nthPage();
        },
        watch: {
            $route() {
                this.nthPage();
            }
        }
    };
</script>
