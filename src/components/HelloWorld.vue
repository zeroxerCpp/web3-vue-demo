<template>
<div>
<el-row :gutter="20">
  <el-col :span="6"><div class="grid-content bg-purple">
    <el-button @click="create()" type="primary">Create Wallet</el-button>
  </div></el-col>
  <el-col :span="6"><div class="grid-content bg-purple">
  </div></el-col>
  <el-col :span="6"><div class="grid-content bg-purple">
    <el-button @click="addFlag = true" type="primary">Add Wallet</el-button>
    <el-dialog title="导入钱包" :visible.sync="addFlag">
  <el-form>
    <el-form-item label="私钥">
      <el-input v-model="account.privateKey" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="addFlag = false">取 消</el-button>
    <el-button type="primary" @click="add()">添加</el-button>
  </div>
</el-dialog>
  </div></el-col>
  <el-col :span="6"><div class="grid-content bg-purple">
    <el-button @click="getBalance()" type="primary">Get Balance</el-button>
  </div></el-col>
  <el-col :span="6"><div class="grid-content bg-purple">
  </div></el-col>
  <el-col :span="6"><div class="grid-content bg-purple">
    <el-button @click="sendFlag = true" type="primary">Send To</el-button>
    <el-dialog title="发送" :visible.sync="sendFlag">
  <el-form>
    <el-form-item label="数量" title="">
      <el-input v-model="sendAmount" autocomplete="off"></el-input>
     <div style="float: left;">余额:{{ account.balance }} </div>
    </el-form-item>
    <el-form-item label="接收地址">
      <el-input v-model="receiveAddr" autocomplete="off"></el-input>
    </el-form-item>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="sendFlag = false">取 消</el-button>
    <el-button type="primary" @click="send(receiveAddr,sendAmount)">提交</el-button>
  </div>
</el-dialog>
  </div></el-col>
</el-row>

<el-row :gutter="20">
    <p>Address:{{ account.address }}</p>
</el-row>
<el-row :gutter="20">
    <p>PrivateKey:{{ account.privateKey }}</p>
</el-row>
<el-row :gutter="20">
    <p>Balance:<div v-loading="transactionLoading">{{ account.balance }} </div></p>
</el-row>
<el-row v-if="hash!='' " :gutter="20">
    <p>TransactionHash:<div v-loading="transactionLoading">
      <a>{{ GOERLI_EXPLORE+hash }}</a> </div></p>
</el-row>
</div>
</template>

<script>
import {RPC_TEST} from "../assets/sources/RPC";
import Web3 from "web3";
var web3= new Web3(Web3.givenProvider || RPC_TEST.GOERLI);
export default {
  data() {
    return {
      GOERLI_EXPLORE:RPC_TEST.GOERLI_EXPLORE,
      hash:'',
      transactionLoading:false,
      sendFlag:false,
      sendAmount:'',
      receiveAddr:'0xdde0b6eec2b51abd36908766fbff9c6b1fcdf2e8',
      addFlag:false,
      GOERLI: RPC_TEST.GOERLI,
      account:{
        address:"",
        privateKey:"0xba7d9ba5a2bf986bac30fa576b4da6fef4da4d4778fff9eec8cd11556c28319d",
        balance:""
      }
    };
  },

  methods: {
   create() {
      const account = web3.eth.accounts.create("123");
      this.account.address = account.address;
      this.account.privateKey = account.privateKey;
      console.log(this.account);
    },
    
       async getBalance() {
       await web3.eth.getBalance(this.account.address,'latest').then((res)=>{
        // this.account.balance = res;
        // 更新响应式数据引用的值
        this.account.balance = web3.utils.fromWei(res,"ether");
        // 输出余额
        console.log(this.account.balance);

      });
      
    },

    add(){  
      if(this.account.privateKey==''){
        console.log(this.account.privateKey.length);
        console.warn("there's error with private key");
      }else{
        const account = web3.eth.accounts.wallet.add(this.account.privateKey);
        this.account.address = account[0].address;
        this.account.privateKey = account[0].privateKey;
        this.getBalance();
        this.addFlag = false;
      }
    },

    async send(anotherAddress,amount) {
      this.sendFlag = false;
      try {
    // 获取 gas 价格
    const gasPrice = await web3.eth.getGasPrice();

    // 构造交易对象
    const tx = {
      from: this.account.address,
      to: anotherAddress,
      value: web3.utils.toWei(amount, 'ether'),
      gasPrice: gasPrice
    };

    // 获取 gas 估算值
    const gas = await web3.eth.estimateGas(tx);

    // 添加 gas 限制到交易对象
    tx.gas = gas;

    // 发送交易并等待结果
    this.transactionLoading = true;
    const receipt = await web3.eth.sendTransaction(tx);
    this.getBalance();
    this.transactionLoading = false;
    // 返回交易哈希
    // console.log(receipt.transactionHash);
    this.hash = receipt.transactionHash;
  } catch (error) {
    console.error('Failed to send transaction:', error);
    throw error;
  }

    },


  }
};




</script>
