<template>
<section>
  <el-row class="tac custom-scrollbar" id="vpic_sidebar">
    <el-col :span="24">

      <el-menu unique-opened default-active="1-2" class="el-menu-vertical-demo">

        <el-submenu index="1">
          <template slot="title"><vicon name="object-ungroup" scale="1.5" class="menu_icon"></vicon>Elements</template>
          
          <el-submenu index="1-1">
            <template slot="title"><vicon name="font" scale="1.5" class="menu_icon"></vicon>Raw text</template>
              <el-menu-item index="1-1-1">
                <div class="menue-checkboxes">
                  <el-checkbox class="indetermined" :indeterminate="isIndeterminateText" style="margin: 15px 0;" v-model="checkAllTexts" @change="handleCheckAllTextsChange">Include all texts</el-checkbox>
                  <el-checkbox-group v-model="checkedTexts" @change="handlecheckedTextsChange">
                    <el-checkbox v-for="text in texts" :label="text" :key="text">{{text}}</el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-menu-item>
          </el-submenu>
          
          <el-submenu index="1-2"><template slot="title"><vicon name="qrcode" scale="1.5" class="menu_icon"></vicon>QR code</template>
              <el-menu-item index="1-2" >
                <div class="menue-checkboxes">
                  <el-checkbox class="indetermined" :indeterminate="isIndeterminateQR" style="margin: 15px 0;" v-model="checkAllQR" @change="handleCheckAllQRChange">Include all QR codes</el-checkbox>
                  <el-checkbox-group v-model="checkedQR" @change="handlecheckedQRChange">
                    <el-tooltip placement="top">
                      <div slot="content">This QR is only working on<br/>upcoming NEMpay mobile app</div>
                      <el-checkbox v-for="qr in experimental" :label="qr" :key="qr" disabled>{{qr}}</el-checkbox>
                    </el-tooltip>
                    <el-checkbox v-for="qr in working" :label="qr" :key="qr">{{qr}}</el-checkbox>
                  </el-checkbox-group>
                </div>
              </el-menu-item>
          </el-submenu>
        </el-submenu>
      </el-menu>  
    </el-col>
  </el-row>
  <el-row type="flex" class="row-bg" justify="center">
    <el-col spane="24">  
      <div class="logo">
        <img src="./assets/img/logo.png" width="100px">
      </div>   
    </el-col>
  </el-row>
</section>
</template>

<script>
import Vicon from './vicon';
import nem from 'nem-sdk';
import QRious from 'qrious';

const Texts = ['PrivateKey', 'Password', 'Wallet Name', 'Address'];
const QRs = ['Mobile Import', 'Voucher', 'NEMpay', 'PrivateKey', 'Password', 'Password & PrivateKey'];

  export default {
    data() {
      return {
        heckAll: true,
        checkedTexts: ['PrivateKey', 'Password'],
        texts: Texts,
        qrs: QRs,
        checkedQR: ['Mobile Import', 'Voucher', 'NEMpay'],
        isIndeterminateText: true,
        isIndeterminateQR: true,
      };
    },
    methods: {
      handleCheckAllTextsChange(event) {
        this.checkedTexts = event.target.checked ? Texts : [];
        this.isIndeterminateText = false;
      },
      handleCheckAllQRChange(event) {
        this.checkedQR = event.target.checked ? QRs : [];
        this.isIndeterminateQR = false;
      },
      handlecheckedTextsChange(value) {
        let checkedCount = value.length;
        this.checkAll = checkedCount === this.texts.length;
        this.isIndeterminateText = checkedCount > 0 && checkedCount < this.texts.length;
      },
      handlecheckedQRChange(value) {
        let checkedCount = value.length;
        this.checkAll = checkedCount === this.qrs.length;
        this.isIndeterminateQR = checkedCount > 0 && checkedCount < this.qrs.length;
      },
    },
    computed: {
      experimental: function(){
          return QRs.filter(function (item) {
          return item === 'NEMpay';
        });
      },
      working: function(){
          return QRs.filter(function (item) {
          return item !== 'NEMpay';
        });
      },
    },
    components: {
      Vicon,
    },
    mounted: function () {
      // generat QR code for mobile import
      // encrypt the private key for mobile usage
      const _this = this;
      let mobileKeys = nem.crypto.helpers.toMobileKey(
                          _this.$store.getters.getPassword, 
                          _this.$store.getters.getPrivateKey);

      // QR structure for mobile import
      let QRMobileObj = {
        "v": this.$store.getters.getNetwork === nem.model.network.data.testnet.id ? 1 : 2,
        "type": 3,
        "data": {
          "name": this.$store.getters.getName,
          "priv_key": mobileKeys.encrypted,
          "salt": mobileKeys.salt
        },
      };

      // QR structure for voucher
      let QRVoucherObj = {
        "v": this.$store.getters.getNetwork === nem.model.network.data.testnet.id ? 1 : 2,
        "type": 1,
        "data": {
          "addr": this.$store.getters.getAddress,
          "name": this.$store.getters.getName,
        },
      };

      //QR structure for NEMpay
      let QRNEMpayObj = {
        'nem': {
          'type': 1,
          'version': 1,
          'name': this.$store.getters.getName,
          'enc_priv': this.$store.getters.getWallet.accounts[0].encrypted,
          'iv': this.$store.getters.getWallet.accounts[0].encrypted.iv,
          'indexes': Object.keys(this.$store.getters.getWallet.accounts).length,
          'accountLabels': []
        }
      };

      // turning the QR objects into strings and committing them
      const QRMobileData = JSON.stringify(QRMobileObj);
      const QRVoucherData = JSON.stringify(QRVoucherObj);
      const QRNEMpayData = JSON.stringify(QRNEMpayObj);

    },
    
    
  };
</script>

<style>

.el-submenu .el-menu-item{
  height: auto;
  white-space: pre-wrap;
}

.submenu__title{
  font-size: 20px;
}
.menu_icon{
  margin-right: 15px;
}
.el-checkbox{
  margin-left: 15px;
}


.el-checkbox__label{
  font-size: 12px;
}

.undetermined{
  font-size: 14px;
}

.logo{
  display: flex;
  flex-direction: column;
  position: fixed;
  height: 200Px;
  bottom: 20px;
  left: 100px;
  align-items: center;
}

.logo img {
  margin: auto;
}


</style>
