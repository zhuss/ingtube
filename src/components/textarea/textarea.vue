<template>
<div class="in-textarea-warp">
    <p class="in-textarea-placeholder" v-show="placeholder&&value==''">{{placeholder}}</p>
    <div class="in-textarea"
         ref="textarea" 
         :contenteditable="!disabled"
         @input="handleInput"
         @focus="handleFocus"
         @blur="handleBlur"></div>
</div>
</template>
<script>
export default {
    name:'InTextarea',
    props:{
        value:{
            type:String
        },
        placeholder:{
            type:String,
            default:''
        },
        disabled:{
            type:Boolean,
            default:false
        }
    },
    data(){
        return {
            isFocus:false,
            oldValue:''
        }
    },
    computed:{
        currentValue(){
            let innerHTML = '';
            if(this.value==''){
                innerHTML = '<p><br></p>';
            }else{
                let valueArray = this.value.split('\n');
                for(let i =0; i < valueArray.length; i++){
                    innerHTML += '<p>'+valueArray[i]+'</p>'
                }
            }
            return innerHTML;
        }
    },
    watch:{
        value(value){
            if(!this.isFocus){
                this.$refs.textarea.innerHTML = this.currentValue;
            }
        }
    },
    mounted(){
        this.oldValue = this.value;
        this.$refs.textarea.innerHTML = this.currentValue;
    },
    methods:{
        handleInput(e){
           let html = e.target.innerHTML;
            //删除到
            if(html===''){
                e.target.innerHTML = '<p><br></p>';
                html = '<p><br></p>';
            }
            this.$emit('input',e.target.innerText.trim('\n\n').replace(/\n\n/g,'\n'));
        },
        handleFocus(e){
            this.isFocus = true;
            this.oldValue = this.value;
            this.$emit('focus',e);
        },
        handleBlur(e){
            this.isFocus = false;
            if(e.target.innerText.trim('\n\n').replace(/\n\n/g,'\n') != this.oldValue){
                let event = {
                    type:'change',
                    target:e.target
                }
                this.$emit('change',event);
            }
            this.$emit('blur',e);
        }
    }
}
</script>


