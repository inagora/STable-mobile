<template>
	<div v-if="btns.length>0" class="st-toolbar">
		<template v-for="(btn, btnIdx) of btns">
			<div v-if="btn=='|'" :key="btnIdx" class="st-toolbar-separator">
				&nbsp;
			</div>
			<div v-else-if="btn==' '" :key="btnIdx" class="st-toolbar-space"></div>
			<x-button
				v-else
				:key="btnIdx"
				ref="btn"
				:conf="btn"
				@click="triggerClick(btn, $event)"
			/>
		</template>
	</div>
</template>

<script>
import XButton from './com/Button.vue';
import {loadJs} from './util/util.js';
import {create} from './com/Dialog.js';
import qtip from './com/qtip';
import XForm from './form/index.vue';

export default {
	components: {XButton},
	inject: {
		title: 'title',
		toolbar: {
			default: []
		},
		addUrl: 'addUrl',
		addConfig: 'addConfig',
		batDeleteUrl: {
			default: ''
		},
		actionMethods: 'actionMethods',
		store: 'store',
		idIndex: 'idIndex',
		ajax: 'ajax',
		locale: 'locale'
	},
	data(){
		let self = this;
		let tb = [];
		if(this.toolbar && Array.isArray(this.toolbar)) {
			tb = Array.from(this.toolbar);
		}
		if(this.addUrl || this.batDeleteUrl) {
			if(tb.length>0)
				tb.unshift('|');
			if(this.batDeleteUrl){
				tb.unshift({
					type: 'danger',
					text: this.locale.toolbar.batDelete,
					icon: 'st-icon st-icon-delete',
					click(){
						self.batDelete();
					}
				});
			}
			if(this.addUrl) {
				tb.unshift({
					type: 'success',
					text: this.locale.add,
					icon: 'st-icon st-icon-plus',
					click(){
						self.add();
					}
				});
			}
		}
		return {
			btns: tb
		};
	},
	mounted(){
		if(this.downloadable) {
			loadJs('https://cdn.jsdelivr.net/npm/xlsx@0.15.0/dist/xlsx.full.min.js');
		}
	},
	methods: {
		triggerClick(btn, evt){
			btn.click&&btn.click.call(this.$parent, btn, evt);
		},
		add(){
			let toolbar = this;
			let html = '<x-form id="_st_add_form" size="medium" :field-list="fields" :default-values="params" label-width="100px" :label-width="100" :action-methods="actionMethods" @submit="create"></x-form>';
			create({
				title: this.locale.add,
				width: 600,
				height: '62%',
				html,
				buttons: [
					{
						text: this.locale.add,
						nativeType: 'submit',
						form: '_st_add_form',
						type: 'success'
					},
					{
						text: this.locale.cancel,
						click(){
							this.close();
						}
					}
				],
				components:{
					XForm
				},
				data: {
					fields: this.addConfig,
					params: {},
					actionMethods: this.actionMethods
				},
				methods: {
					create(data){
						let ret = toolbar.store.emit('beforeadd', data);
						if(ret===false)
							return;
						let addUrl = toolbar.addUrl;
						if(ret) {
							if(ret.url)
								addUrl = ret.url;
							if(ret.data)
								data = ret.data;
						}
						toolbar.ajax.request({url: addUrl, data, method:toolbar.actionMethods.create}).then(res=>{
							if(res.errno==0){
								qtip.success(toolbar.locale.toolbar.addSuccessMsg);
								this.destroy();
								toolbar.store.$emit('load');
								toolbar.store.emit('add', res, data);
							} else {
								qtip.error(res.errmsg);
							}
						});
					}
				}
			});
		},
		batDelete(){
			let records = this.$parent.getSelected();
			if(!records || records.length<=0){
				qtip.error(this.locale.toolbar.deleteTips);
				return;
			} else if(!this.idIndex){
				qtip.error(this.locale.noIdindex);
				return;
			}
			let ret = this.store.emit('beforebatdelete', records);
			if(ret===false){
				return;
			}
			let ids = records.map(r=>r[this.idIndex]);
			
			this.ajax.request({
				url: this.batDeleteUrl,
				data: {
					[this.idIndex]: ids
				},
				method: this.actionMethods.delete
			}).then(res=>{
				if(res.errno){
					qtip.error(res.errmsg);
				} else {
					qtip.success(this.locale.toolbar.deleteSuccessMsg);
					this.store.emit('batdelete', res, records);
				}
			});
		},
		download(){
			let records = this.store.getCurrentPage();
			let ret = this.store.emit('beforeexport', records);
			if(ret===false)
				return;
			if(Array.isArray(ret)) {
				records = ret;
			}
			this.export(records);
		},
		downloadAll(){
			this.store.getAllPages().then(records=>{
				let ret = this.store.emit('beforeexport', records);
				if(ret===false)
					return;
				if(Array.isArray(ret)) {
					records = ret;
				}
				this.export(records);
			});
		},
		export(records){
			let table = this.$parent.$refs.table[0];
			let columns = [].concat(table.leftColumns).concat(table.freeColumns).concat(table.rightColumns);
			columns = columns.filter(col=>['text','render'].includes(col.type));

			let headers = [];
			let colsConf = [];
			let sheetData = [];
			columns.forEach(col=>{
				headers.push(col.text);
				colsConf.push({
					wpx: col._st_width
				});
			});
			sheetData.push(headers);

			for(let record of records) {
				let row = [];
				for(let col of columns){
					let idx = col.dataIndex;
					let val = record[idx];

					if(col.type=='render'){
						val = record._st_aux.render[idx];
						if(typeof val=='string')
							val = val.replace(/(<([^>]+)>)/ig,"").trim();
					}
					if(typeof val!='number' && col.exportType=='number'){
						if(typeof val=='string'){
							val = val.replace(/,/g, '');
						}
						val = Number(val);
					}
					row.push(val);
				}
				sheetData.push(row);
			}

			let wb = XLSX.utils.book_new();
			let name = prompt(this.locale.toolbar.confirmFileName,this.title||'data');
			name = (name||'stable').trim();
			if(!name.includes('.')) {
				name += '.xlsx';
			}
			let ws = XLSX.utils.aoa_to_sheet(sheetData);
			ws['!cols'] = colsConf;
			XLSX.utils.book_append_sheet(wb, ws, 'Sheet1');
			XLSX.writeFile(wb, name);
		}
	}
};
</script>

<style lang="scss">
.st-toolbar{
	border-bottom: 1px solid #d0d0d0;
	padding: 10px 0 0 10px;
	display: flex;
	flex-wrap: wrap;
	align-items: center;

	&-separator{
		display: inline-block;
		width: 1px;
		overflow: hidden;
		margin-right: 8px;
		margin-bottom: 10px;
		background-color: #d0d0d0;
	}
	&-space{
		width: 10px;
	}

	.st-btn{
		margin-right: 8px;
		margin-bottom: 10px;
	}

}
</style>
