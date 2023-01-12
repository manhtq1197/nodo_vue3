<template>
  <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="120px" class="content-form">
    <el-row>
      <el-col :span="24">
        <el-form-item label="Insurance type" prop="insuranceType" required>
          <el-select v-model="ruleForm.insuranceType" placeholder="Select item" style='width: 330px'>
            <el-option label="Short term insurance" value="short"></el-option>
            <el-option label="Annual insurance" value="annual"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Period start" prop="periodStart" required>
          <el-date-picker :disabledDate="disabledPeriodStart" type="date" placeholder="Pick a date"
                          v-model="ruleForm.periodStart" style='width: 330px'></el-date-picker>
        </el-form-item>
        <el-form-item label="Period end" prop="periodEnd"
                      :required="ruleForm.insuranceType === 'short'"
                      :rules="[{required: ruleForm.insuranceType === 'short',
                                     message: 'Please pick period end', trigger: 'blur'}]">
          <el-date-picker :disabledDate="disabledPeriodEnd" type="date" placeholder="Pick a date"
                          v-model="ruleForm.periodEnd" style='width: 330px'></el-date-picker>
        </el-form-item>
        <el-form-item label="Package" prop="package" required>
          <el-radio-group v-model="ruleForm.package">
            <el-radio v-for="(item, index) in packageData" :key="index" :label="item.type"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="Additional charges" prop="additionalCharges">
          <el-radio-group v-model="ruleForm.additionalCharges">
            <el-radio v-for="(item, index) in chargesData" :key="index" :label="item.type"></el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="No. of persons" prop="noOfPersons">
          <el-input type="number" v-model="ruleForm.noOfPersons" style='width: 330px'></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm('ruleForm')">Calculation</el-button>
        </el-form-item>
        <el-form-item label="Price of insurance" prop="results">
          <el-input disabled v-model="ruleForm.results" style='width: 330px'></el-input>
        </el-form-item>
      </el-col>
    </el-row>
  </el-form>
</template>
<script>
export default {
  data() {
    const validYearProduct = (rule, value, callback) => {
      if (value && Number.parseInt(value) > 3) {
        callback(new Error('No. of persons should not be more than 3'))
      }
      callback()
    };
    return {
      packageData: [
        {type: 'Basic', short: 1.2, annual: 39},
        {type: 'Extended', short: 1.8, annual: 49},
        {type: 'Extra', short: 2.4, annual: 59},
      ],
      chargesData: [
        {type: 'Cancellation', short: 1.5, annual: 1.2},
        {type: 'Sport activities', short: 1.3, annual: 1.1},
      ],
      ruleForm: {
        insuranceType: '',
        periodStart: '',
        periodEnd: '',
        additionalCharges: '',
        package: '',
        noOfPersons: '',
        results: ''
      },
      rules: {
        insuranceType: [
          {required: true, message: 'Please select Insurance type', trigger: 'change'}
        ],
        periodStart: [
          {type: 'date', required: true, message: 'Please pick period start', trigger: 'change'}
        ],
        package: [
          {required: true, message: 'Please select package', trigger: 'change'}
        ],
        noOfPersons: [
          {required: true, message: 'Please input No. of persons', trigger: 'blur'},
          {validator: validYearProduct, trigger: 'blur'}
        ]
      },
    };
  },
  methods: {

    async submitForm(formName) {
      this.$refs[formName].validate(async (valid) => {
        if (valid) {
          let calPackage = 1
          let calCharges = 1
          await this.packageData.forEach(item => {
            if (item.type === this.ruleForm.package) {
              if (this.ruleForm.insuranceType === 'short') {
                calPackage = ((this.ruleForm.periodEnd - this.ruleForm.periodStart) / 86400000) * item.short
              } else if (this.ruleForm.insuranceType === 'annual') {
                console.log('annual', item.annual)
                calPackage = item.annual
              }
            }
          })
          await this.chargesData.forEach(data => {
            if (data.type === this.ruleForm.additionalCharges) {
              if (this.ruleForm.insuranceType === 'short') {
                calCharges = data.short * this.ruleForm.noOfPersons
              } else if (this.ruleForm.insuranceType === 'annual') {
                calCharges = data.annual * this.ruleForm.noOfPersons
              }
            }
          })
          this.ruleForm.results = new Intl.NumberFormat('de-DE', {
            style: 'currency',
            currency: 'EUR'
          }).format(calPackage * calCharges)
        } else {
          return false;
        }
      });
    },
    disabledPeriodStart(time) {
      if (this.ruleForm.periodEnd) return time.getTime() > this.ruleForm.periodEnd
    },
    disabledPeriodEnd(time) {
      if (this.ruleForm.periodStart) return time.getTime() < this.ruleForm.periodStart
    },
  }
}
</script>
<style>
.content-form {
  padding: 42px 24px 50px;
}
</style>