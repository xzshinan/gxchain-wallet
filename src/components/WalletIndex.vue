<template>
    <div class="page-group">
        <div class="page" id="page-wallet-index">
            <div ref="bar" class="bar bar-nav buttons-fixed" style="background: rgba(37, 40, 113, 0)">
                <a v-if="isNative && channel !== 'blockcity'" href="javascript:;" class="icon gxicon gxicon-scan pull-left" @click="openQRScaner">
                    <input ref="qrfile" @change="onFileUpload" v-if="!isNative" type="file" class="file-selector"/>
                </a>
                <a v-if="isNative && channel === 'blockcity'" href="javascript:;" @click="backToBlockCity" class="icon icon-left"></a>
            </div>
            <div ref="bg" id="bg"></div>
            <wallet-tab></wallet-tab>
            <div class="content pull-to-refresh-content" ref="content">
                <div class="row-top" ref="top" :class="{ios:$route.query.platform=='ios'}">
                    <div class="pull-to-refresh-layer">
                        <div class="preloader">
                            <div class="line-scale">
                                <div></div>
                                <div></div>
                                <div></div>
                                <div></div>
                                <div></div>
                            </div>
                        </div>
                    </div>
                    <div class="loading" v-if="wallets.length==0">
                        <div class="line-scale">
                            <div></div>
                            <div></div>
                            <div></div>
                            <div></div>
                            <div></div>
                        </div>
                    </div>
                    <!--wallets-->
                    <swiper :options="swiperOption" :not-next-tick="true" ref="mySwiper" v-show="wallets.length>0">
                        <swiper-slide v-for="wallet in wallets" :key="wallet.name">
                            <div class="wallet-info">
                                <div class="text-center">
                                    <account-image :account="wallet.account" :size="30"></account-image>
                                    <div>
                                        <a @click="openQRCodeModal" href="javascript:;" class="link-qrcode">{{wallet.account}}&nbsp;
                                            <small class="icon icon-code"></small>
                                        </a>
                                    </div>
                                    <p class="bak-tip">
                                        <router-link :to="linkBackup(wallet.account)" class="btn-backup"
                                                     :class="{visible:!wallet.backup_date}" href="javascript:;">
                                            <small>{{$t('index.backup_wallet')}}&nbsp;
                                                <span class="red-dot"></span>
                                                <span class="icon icon-right"></span>
                                            </small>
                                        </router-link>
                                    </p>
                                </div>
                                <div class="row-balance">
                                    <div>
                                        <div>
                                            <small>{{$t('index.value')}}</small>
                                        </div>
                                        <div>
                                            <small>≈</small>
                                            <span class="digit balance">{{currentWallet.totalValue !== undefined ? currentWallet.totalValue : '-' | number(2)}}</span>
                                        </div>
                                    </div>
                                    <div class="asset-button" @click="goAddAssets">
                                        <!-- <router-link :to="link('/add-assets')" href="javascript:;">
                                            <img src="../assets/images/asset_btn.png">
                                        </router-link> -->
                                        <img src="../assets/images/asset_btn.png">
                                    </div>
                                </div>
                            </div>
                        </swiper-slide>
                        <div class="swiper-pagination" slot="pagination" v-show="wallets&&wallets.length>1"></div>
                    </swiper>
                    <router-link :to="link('/staking-index')" class="link-vote-index">
                        <img src='../assets/images/vote_icon.png'>
                    </router-link>

                    <router-link :to="link('/loyalty-program')" class="link-loyalty-program" v-if="showLoyaltyProgram">
                        <img :src="`static/${$t('index.loyalty_program_icon')}`">
                    </router-link>
                </div>
                <!--function entries-->
                <div class="row-operation">
                    <router-link :to="link('/transfer')" class="item" href="javascript:;">
                        <span class="gxicon x2 gxicon-transfer"></span>
                        <span class="item-text">{{$t('index.transfer')}}</span>
                    </router-link>
                    <div class="sep"></div>
                    <a @click="openQRCodeModal" class="item" href="javascript:;">
                        <span class="gxicon x2 gxicon-collect"></span>
                        <span class="item-text">{{$t('index.receive')}}</span>
                    </a>
                </div>
                <div class="tab-nav index-tab">
                    <div :class="tabIndex === 'tab-container1' ? 'tab-nav-item active' : 'tab-nav-item'" @click="tabIndex = 'tab-container1'">{{$t('index.assets')}}</div>
                    <div :class="tabIndex === 'tab-container2' ? 'tab-nav-item active' : 'tab-nav-item'" @click="tabIndex = 'tab-container2'">{{$t('index.collection')}}</div>
                </div>
                <div class="tab-container">
                    <gxb-tab-container v-model="tabIndex">
                        <gxb-tab-container-item id="tab-container1">
                            <!--assets-->
                            <div class="table-assets" v-if="currentWallet">
                                <div class="list-block media-list">
                                    <ul>
                                        <li v-for="balance in currentWallet.balances" class="item-content item-asset"
                                            :key="balance.symbol">
                                            <div class="item-inner">
                                                <div class="symbol">
                                                    <account-image :account="balance.symbol" :size="15"></account-image>
                                                    <div>&nbsp;&nbsp;{{balance.symbol}} </div>
                                                </div>
                                                <div class="price">
                                                    <div class="digital">
                                                        {{balance.amount | number(balance.precision)}} <span v-if="balance.asset_id == '1.3.1'&&stakingAmount" class="stakingAmount">({{$t('index.staking')}}: {{stakingAmount}}GXC)</span>
                                                    </div>
                                                    <small v-if="balance.value != 0">
                                                        ≈&nbsp;{{$t('index.unit')}}{{balance.value | number(2)}}
                                                    </small>
                                                </div>
                                            </div>
                                        </li>
                                    </ul>
                                </div>
                            </div>
                        </gxb-tab-container-item>
                        <gxb-tab-container-item id="tab-container2">
                           <!-- <div class="table-assets" v-if="accountNFT.length>0">-->
                                <div class="list-block media-list nft-list">
                                        <ul>
                                            <li  class="item-content item-asset">
                                                <div class="item-inner" @click="showNFTGroup(1)">
                                                    <div class="symbol">
                                                        <img src="https://static.gxb.io/gxs/symbols/gxs.png" width="30" height="30">
                                                    </div>
                                                    <div class="price">
                                                        <div class="digital">
                                                            <small>
                                                                GXC官方系列
                                                            </small>
                                                        </div>
                                                    </div>
                                                </div>
                                            </li>
                                            <li  class="item-content item-asset">
                                                <div class="item-inner" @click="showNFTGroup(2)">
                                                    <div class="symbol">
                                                        <img src="http://static.gxcfly.top/images/logogxc-fly-logo.png"  width="32" height="32">
                                                    </div>
                                                    <div class="price">
                                                        <div class="digital">
                                                            <small>
                                                                FLY定制系列
                                                            </small>
                                                        </div>
                                                    </div>
                                                </div>
                                            </li>
                                            <li  class="item-content item-asset">
                                                <div class="item-inner" @click="showNFTGroup(3)">
                                                    <div class="symbol">
                                                        <img src="https://static.gxb.io/dapp/blockcity/nodevote/20.png"  width="32" height="32">
                                                    </div>
                                                    <div class="price">
                                                        <div class="digital">
                                                            <small>
                                                                MOON自由创作系列
                                                            </small>
                                                        </div>
                                                    </div>
                                                </div>
                                            </li>
                                    </ul>
                                </div>
                  <!--          <div v-else class="no-reocrd table-assets">
                                {{$t('node_vote.index.no_record')}}
                            </div>-->
                        </gxb-tab-container-item>
                    </gxb-tab-container>
                </div>

            </div>
        </div>
        <left-panel ref="panel"></left-panel>
        <account-q-r-code ref="qrcode" :account="this.currentWallet.account"></account-q-r-code>
    </div>
</template>
<script>
    import {swiper, swiperSlide} from 'vue-awesome-swiper';
    import AccountImage from './sub/AccountImage.vue';
    import LeftPanel from './sub/LeftPanel.vue';
    import AccountQRCode from './sub/AccountQRCode.vue';
    import util from '@/common/util';
    import WalletTab from './sub/WalletTab';

    import {
        fetch_account_balances, fetch_account, get_staking_object, fetch_reference_accounts, get_assets_by_ids, get_wallet_index, get_wallets,
        set_wallet_index
    } from '@/services/WalletService';
    import {get_market_asset_price} from '@/services/MarketService';
    import filters from '@/filters';
    import some from 'lodash/some';
    import unionBy from 'lodash/unionBy';
    import {
        mapActions
    } from 'vuex';
    export default {
        filters,
        data () {
            let self = this;
            let wallets = get_wallets();
            wallets = wallets.map((w) => {
                w.balances = [];
                return w;
            });
            let wallet_index = get_wallet_index();
            return {
                accountCopied: false,
                wallets: wallets,
                currentWalletIndex: wallet_index,
                swiperOption: {
                    pagination: '.swiper-pagination',
                    autoplay: false,
                    keyboardControl: true,
                    paginationClickable: true,
                    initialSlide: wallet_index,
                    onSlideChangeEnd: function (s) {
                        let wallets = get_wallets();
                        let wallet = wallets[s.activeIndex];
                        self.isAssetHidden = wallet.isAssetHidden;
                        set_wallet_index(s.activeIndex);
                        self.currentWalletIndex = s.activeIndex;
                    }
                },
                isAssetHidden: '',
                channel: '',
                stakingAmount: 0,
                tabIndex: 'tab-container1',
                nftBalance: [{'asset_id': '1.3.1', 'amount': 1052.16472, 'precision': 5, 'value': 2413.77, 'symbol': 'GXC', 'status': true}],
                accountNFT: [],
                showNft: false,
                currentNft: {}
            };
        },
        watch: {
            currentWalletIndex () {
                this.loadBalances(this.currentWallet);
                this.getStakingAmount();
            }
        },

        methods: {
            ...mapActions({
                setAccountNft: 'setAccountNft'
            }),
            showNFTInfo (item) {
                let query = {
                    from: this.$route.fullPath
                };
                this.$router.push({
                    path: this.link(`/nftInfo/${item.id}`, query)
                });
            },
            showNFTGroup (index) {
                const item = [process.env.nftContract, process.env.flyContract, process.env.moonContract][index - 1];
                this.$router.push({
                    path: this.link(`/nftGroup/${item}`)
                });
            },
            loadWallets () {
                if (this.wallets.length == 0) {
                    this.$router.replace({
                        path: this.link('/wallet-create')
                    });
                } else {
                    this.wallets.forEach((wallet) => {
                        this.loadBalances(wallet);
                    });
                    this.getStakingAmount();
                }
                setTimeout(() => {
                    $.pullToRefreshDone($(this.$el).find('.pull-to-refresh-content'));
                }, 500);
            },
            loadBalances (wallet) {
                if (wallet.account) {
                    let wallet_balances;
                    let assetMap = {};
                    let priceMap = {};
                    wallet.totalValue = '-';
                    fetch_account_balances(wallet.account).then(function (balances) {
                        if (!balances) {
                            return;
                        }
                        wallet_balances = balances;
                        let asset_ids = wallet_balances.map(b => {
                            return b.asset_id;
                        });
                        return get_assets_by_ids(asset_ids);
                    }).then(assets => {
                        let priceSymbol = '';
                        assets.forEach(asset => {
                            assetMap[asset.id] = asset;
                            priceSymbol += asset.symbol + ',';
                        });
                        priceSymbol = priceSymbol.substring(0, priceSymbol.length - 1);
                        wallet.balances = wallet_balances.map(b => {
                            return {
                                asset_id: b.asset_id,
                                amount: b.amount / Math.pow(10, assetMap[b.asset_id].precision),
                                precision: assetMap[b.asset_id].precision,
                                symbol: assetMap[b.asset_id].symbol,
                                value: 0
                            };
                        });
                        return get_market_asset_price(priceSymbol);
                    }).then(prices => {
                        wallet.totalValue = 0;
                        prices.forEach(price => {
                            priceMap[price.symbol] = price;
                        });
                        wallet.balances = wallet_balances.map(b => {
                            let assetValue = priceMap[assetMap[b.asset_id].symbol] ? b.amount / Math.pow(10, assetMap[b.asset_id].precision) * priceMap[assetMap[b.asset_id].symbol].price : 0;
                            wallet.totalValue += assetValue;
                            return {
                                asset_id: b.asset_id,
                                amount: b.amount / Math.pow(10, assetMap[b.asset_id].precision),
                                precision: assetMap[b.asset_id].precision,
                                symbol: assetMap[b.asset_id].symbol,
                                value: assetValue
                            };
                        });
                        return wallet.balances;
                    }).then((balances) => {
                        let wallet_balances = balances.map(b => {
                            return {
                                asset_id: b.asset_id,
                                amount: b.amount,
                                precision: b.precision,
                                value: b.value,
                                symbol: b.symbol,
                                status: true
                            };
                        });
                        // 将当前钱包账号已有资产存入localStorage
                        let assetsArray = localStorage.getItem('assets_array');
                        if (!assetsArray) {
                            assetsArray = [];
                        } else {
                            assetsArray = JSON.parse(assetsArray);
                        }
                        let asset = {
                            account: wallet.account,
                            list: wallet_balances.filter((item) => {
                                return item.symbol != 'GXS' && item.symbol != 'GXC';
                            })
                        };
                        let alreadyExist = some(assetsArray, (item) => {
                            return item.account == wallet.account;
                        });

                        if (alreadyExist) {
                            assetsArray = assetsArray.map((item) => {
                                if (item.account == wallet.account) {
                                    item.list.forEach((i) => {
                                        wallet_balances = wallet_balances.map(b => {
                                            if (i.symbol == b.symbol) {
                                                b.status = i.status;
                                            }
                                            return b;
                                        });
                                    });
                                    let list = unionBy(wallet_balances, item.list, 'symbol');
                                    wallet.balances = list.filter((item) => {
                                        return item.status == true;
                                    });
                                    item.list = list.filter((item) => {
                                        return item.symbol != 'GXS' && item.symbol != 'GXC';
                                    });
                                }
                                return item;
                            });
                        } else {
                            wallet.balances = wallet_balances;
                            assetsArray.push(asset);
                        }
                        localStorage.setItem('assets_array', JSON.stringify(assetsArray));
                    }).catch(ex => {
                        console.error(ex);
                    });
                }
            },
            getStakingAmount () {
                let currentWallet = this.currentWallet.account;
                fetch_account(currentWallet).then(result => {
                    this.currentAccountId = result.id;
                    get_staking_object(result.id).then(res => {
                        let stakingList = res;
                        let amount = 0;
                        for (let i = 0; i < stakingList.length; i++) {
                            amount = util.accAdd(amount * 1, stakingList[i].amount.amount * 1);
                        }
                        this.stakingAmount = util.accDiv(amount, 100000);
                    });
                });
            },
            getMarketCap (wallet) {
                let balance = Number(wallet.balance);
                let price = this.price;
                if (!isNaN(balance) && price != '-') {
                    return wallet.balance * this.price;
                }
                return '-';
            },
            openPanel () {
                this.$refs.panel.open();
            },
            openQRScaner () {
                let self = this;
                if (this.isNative) {
                    util.callNativeForWebView(function (result) {// eslint-disable-line
                        let query = {
                            to: result
                        };
                        if (result.indexOf('qr://transfer') == 0) {
                            query = util.query2Obj(result.replace('qr://transfer', ''));
                        }
                        self.$router.push({
                            path: self.link('/transfer', query)
                        });
                    }, null, 'QRCode', 'scan', []);
                }
            },
            linkBackup (account) {
                let query = {
                    account: this.currentWallet.account,
                    from: this.$route.fullPath
                };
                return this.link('/wallet-backup-detail', query);
            },
            openQRCodeModal () {
                this.$refs.qrcode.show();
            },
            goAddAssets () {
                let query = {
                    account: this.currentWallet.account,
                    from: this.$route.fullPath
                };
                this.$router.push({
                    path: this.link('/add-assets', query)
                });
            },
            backToBlockCity () {
                if (this.isNative) {
                    util.callNativeForWebView(null, null, 'Controller', 'pop', []); //eslint-disable-line
                }
            }
        },
        destroyed () {
            $.allowPanelOpen = true;
            $(this.$el).off('refresh');
        },
        mounted () {
            if (this.$route.query.channel) {
                this.channel = this.$route.query.channel;
            }
            $.init();
            this.loadWallets();
            fetch_reference_accounts(this.wallets.filter(wallet => {
                return !wallet.partial;
            }).map(wallet => {
                return wallet.account;
            })).then((wallets) => {
                if (wallets.length > this.wallets.length) {
                    location.reload();
                }
            }).catch(ex => {
                console.error(ex);
            });
            this.isAssetHidden = this.currentWallet.isAssetHidden;
            if (this.isNative) {
                window.webview = {
                    reload: () => {
                        this.loadWallets();
                    }
                };
                util.callNativeForWebView(null, null, 'statusBar', 'styleLightContent', []);// eslint-disable-line
            }
            $(this.$el).on('refresh', '.pull-to-refresh-content', (e) => {
                this.loadWallets();
            });
            $(this.$refs.content).on('scroll', () => {
                let offset = $(this.$el).find('.row-top').offset().top;
                if (offset < -20) {
                    let percent = Math.min((-40 - offset) / 40, 1);
                    $(this.$refs.bar).css({
                        background: `rgba(37, 40, 113, ${percent})`
                    });
                } else {
                    $(this.$refs.bar).css({
                        background: 'rgba(37, 40, 113, 0)'
                    });
                }
            }).on('touchmove', (e) => {
                $(this.$refs.bg).css({
                    height: Math.max($(this.$refs.top).height() + $(this.$refs.top).offset().top, $(
                        this.$refs.top).height()),
                    transition: 'none',
                    webkitTransition: 'none'
                });
                return true;
            }).on('touchend', () => {
                const end = () => {
                    $(this.$refs.bg).attr('style', '');
                    clearTimeout(timer);
                };
                let timer = setTimeout(end, 1000);
            });
            this.swiper.update();
        },
        computed: {
            showLoyaltyProgram () {
                return localStorage.getItem('version') != '1.2.0';
            },
            qrcode () {
                return `qr://transfer?account=${this.currentWallet.account}`;
            },
            currentWallet () {
                if (this.wallets.length > 0) {
                    return this.wallets[this.currentWalletIndex];
                }
                return {};
            },
            price () {
                if (!this.exchanges || this.exchanges.length == 0) {
                    return '-';
                } else {
                    let best_price = 0;
                    this.exchanges.forEach((exchange) => {
                        if (this.$i18n.locale == 'zh-CN') {
                            best_price = Math.max(Number(exchange.price_rmb), best_price);
                        } else {
                            best_price = Math.max(Number(exchange.price_dollar), best_price);
                        }
                    });
                    return best_price;
                }
            },
            swiper () {
                return this.$refs.mySwiper.swiper;
            }
        },
        components: {
            swiper,
            swiperSlide,
            AccountImage,
            LeftPanel,
            AccountQRCode,
            WalletTab
        }
    };
</script>
<style lang="scss" scoped="">
    @import "../assets/styles/components/refresh_layer";
    /*::-webkit-scrollbar {*/
    /*display: none;*/
    /*}*/

    #bg {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        background: linear-gradient(to bottom, #150838 0%, #3853bc 49%, #18bce2 100%);
        height: 16.75rem;
        transition: height .2s;
    }

    #page-wallet-index {
        .content {
            top: 0;
            -webkit-overflow-scrolling: auto;
        }
    }

    .bar-nav.buttons-fixed .icon {
        font-size: 1.2rem;
        color: #fff;
    }

    .row-top {
        background: transparent; //linear-gradient(to bottom, #150838 0%, #3853bc 49%, #18bce2 100%);
        color: #fff;
        min-height: 14.8rem;
        display: flex;
        position: relative;
        align-items: center;
        justify-content: center;
        .loading {
            position: absolute;
        }
        &.ios {
            padding-top: 5px;
        }
        .icon {
            font-size: 1.2rem;
            color: #fff;
        }
        .swiper-container {
            width: 100%;
            display: block;
        }
    }

    .wallet-info {
        margin-top: 3.5rem;
        .account-header {
            display: inline-block;
        }
        .link-qrcode {
            color: #fff;
            .icon {
                position: relative;
                top: -1px;
                font-size: .8rem;
            }
        }
        .bak-tip {
            margin: .4rem 0;
        }
        .btn-backup {
            visibility: hidden;
            color: #fff;
            .red-dot {
                position: relative;
                display: inline-block;
                top: -8px;
                border-radius: 2px;
                width: 4px;
                height: 4px;
                background: #ff6e35;
            }
            .icon-right {
                position: relative;
                top: -2px;
                font-size: .6rem;
                margin-left: .2rem;
            }
        }

        .row-balance {
            display: flex;
            justify-content: space-between;
            .asset-button {
                padding: .4rem 0;
                img {
                    width: 1.4rem;
                }
                display: flex;
                flex-direction: column;
                justify-content: flex-end;
            }
            small {
                font-size: .6rem;
            }
            padding: 0 .75rem;
            .digit {
                font-family: bebas;
                font-size: 1.1rem;
            }
        }
    }

    .row-operation {
        height: 3.6rem;
        display: flex;
        padding: 0.75rem;
        background: #fff;
        position: relative;
        z-index: 1;
        border-bottom: .2rem solid #e7e7e7;
        .item {
            display: flex;
            flex: 2;
            font-weight: bold;
            flex-direction: row;
            justify-content: center;
            align-items: center;
            .gxicon {
                width: 35px;
                margin-right: 1.25rem;
            }
        }

        .sep {
            display: flex;
            height: 100%;
            width: 1px;
            background: #eee;
        }
    }

    .pull-to-refresh-layer {
        position: absolute;
        top: 1rem;
        .line-scale > div {
            background-color: #fff !important;
        }
    }

    .link-vote-index {
        z-index: 2;
        position: absolute;
        top: 5.5rem;
        left: 1rem;
        width: 1.6rem;
        height: 1.6rem;
        img {
            width: 100%;
            height: 100%;
        }
    }

    .link-loyalty-program {
        z-index: 2;
        position: absolute;
        top: 5.5rem;
        right: 0;
        width: 4.1rem;
        height: 1.6rem;
        img {
            width: 100%;
            height: 100%;
        }
    }
    .index-tab {
    }
    .list-block.nft-list{
        margin:0 auto;
        .item-inner {
            display: flex;
        }
    }
    .table-assets {
        position: relative;
        min-height: 25rem;
        background: #fff;
        .list-block {
            background: #e7e7e7;
            margin-top: 0;
            margin-bottom: 0;
            .item-asset {
                .item-inner {
                    display: flex;
                    flex-direction: row;
                    padding-top: .75rem;
                    padding-bottom: .65rem;
                    .symbol {
                        display: flex;
                        flex-direction: row;
                        align-items: center;
                        color: #3d3d3b;
                    }
                    .price {
                        text-align: right;
                        color: #3d3d3b;
                        small {
                            color: #888
                        }
                    }
                    .stakingAmount{
                        font-size: .6rem;

                    }
                }
            }
        }
    }
    .tab-nav {
        background-color: #fff;
        position: relative;
        height: 2rem;
        width: 100%;
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        .tab-nav-item {
            flex: 1;
            height: 2rem;
            line-height: 2rem;
            text-align: center;
            font-size: .65rem;
            border-bottom: 1px solid #f2f2f2;
        }
        .tab-nav-item.active {
            color: #6699ff;
            font-weight: bold;
            border-bottom: 1px solid #6699ff;
        }
    }
    .tab-container{
        background-color: #fff;
    }
    .no-reocrd {
        margin: 1.5rem 0;
        font-size: .7rem;
        color: #80848f;
        text-align: center;
    }
</style>
