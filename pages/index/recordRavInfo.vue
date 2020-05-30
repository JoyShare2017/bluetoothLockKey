<template>
	<view >
		{{theData}}
	</view>
</template>

<script>
	export default{
		data(){
			return{
				theData:''
			}
		},
		onLoad(eee) {
			this.theData=eee.data;
		}
	}
</script>

<style>
</style>
