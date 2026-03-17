<script setup name="staffadd">
import { ref, reactive, onMounted } from 'vue';
import staffHttp from '@/api/staffHttp';
import { useRouter } from 'vue-router';
import OAMain from "@/components/OAMain.vue"
import {useAuthStore} from "@/stores/auth"
import { ElMessage } from 'element-plus';


const router = useRouter();
const authStore = useAuthStore()

let departments = ref([])
let staffForm = reactive({
    email: "",
    password: "",
    realname: "",
    department_id: null,
});
const formRef = ref()
let rules = reactive({
    email: [{required: true, message: "Please enter the email address.", trigger: 'blur'}],
    password: [{required: true, message: "Please enter the password.", trigger: 'blur'}],
    realname: [{required: true, message: "Please enter the real name.", trigger: 'blur'}],
    department_id: [{required: true, message: "Please select a department.", trigger: 'change'}],
})

onMounted(async () => {
    if (authStore.user.is_superuser) {
        try {
            departments.value = await staffHttp.getAllDepartment()
        } catch (detail) {
            ElMessage.error(detail)
        }
    }
})

const onSubmit = () => {
    formRef.value.validate(async (valid, fields) => {
        if(valid){
            try{
                const dept_id = authStore.user.is_superuser
                    ? staffForm.department_id
                    : (authStore.user.department?.id || null)
                await staffHttp.addStaff(staffForm.realname, staffForm.email, staffForm.password, dept_id)
                ElMessage.success('Employee added successfully.')
                router.push({name: 'staff_list'})
            }catch(detail){
                ElMessage.error(detail)
            }
        }
    })
}

</script>

<template>
    <OAMain title="Add Employee">
        <el-card shadow="always">
            <el-form :rules="rules" :model="staffForm" ref="formRef" label-width="120px">
                <el-form-item label="Name" prop="realname">
                    <el-input v-model="staffForm.realname" placeholder="Please enter the name">
                    </el-input>
                </el-form-item>

                <el-form-item label="Email" prop="email">
                    <el-input v-model="staffForm.email" placeholder="Please enter the email"> </el-input>
                </el-form-item>

                <el-form-item label="Password" prop="password">
                    <el-input v-model="staffForm.password" placeholder="Please enter the password" type="password">
                    </el-input>
                </el-form-item>

                <el-form-item label="Department" prop="department_id">
                    <el-select
                        v-if="authStore.user.is_superuser"
                        v-model="staffForm.department_id"
                        placeholder="Please select a department"
                        style="width: 100%"
                    >
                        <el-option
                            v-for="dept in departments"
                            :key="dept.id"
                            :label="dept.name"
                            :value="dept.id"
                        />
                    </el-select>
                    <el-input v-else readonly disabled :value="authStore.user.department?.name" placeholder="Department" />
                </el-form-item>

                <el-form-item label="Leader">
                    <el-input readonly disabled :placeholder="'[' + authStore.user.email + ']' + authStore.user.realname">
                    </el-input>
                </el-form-item>

                <el-form-item>
                    <el-button type="primary" @click="onSubmit"> Submit </el-button>
                </el-form-item>
            </el-form>
        </el-card>
    </OAMain>
</template>

<style scoped></style>