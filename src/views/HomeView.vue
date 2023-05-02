<template>
	<div class="home">
		<h2>Тестовое задание FrontEnd</h2>
		<div>
			
			<h3>1. Поле Link</h3>
			<div v-if="!editing">
				<div>
					<a :href="url" target="_blank">{{ title ? title : url }}</a>
				</div>
				<Button @click="startEditing">Edit</Button>
			</div>
			<div v-else>
				<InputText type="text" v-model="editedURL"/>
				<Button @click="saveEdit">Save</Button>
				<Button @click="cancelEdit">Cancel</Button>
			</div>
		</div>
		<div>
			
			<h3>2. DateTime picker </h3>
			<div>
				<Calendar id="calendar-timeonly"
						  v-model="time" 
						  timeOnly 
						  format="HH:mm"/>
				<p>{{ formattedTime }}</p>
			</div>
		</div>
		
		<div>
			<h3>3. Сброс значений в Select</h3>
			<MultiSelect v-model="selectedItems" 
						 :options="options" 
						 optionLabel="name" 
						 placeholder="Select Cities"
						 :maxSelectedLabels="3" 
						 class="w-full md:w-20rem" />
		</div>
		
		<div><h3>4. Многострочный текст в ячейке</h3>
			<div>
<!--				<DataTable :value="data">-->
<!--					<Column field="log" header="Лог">-->
<!--						<template #body="slotProps">-->
<!--							<div v-for="(line, index) in splitLog(slotProps.rowData.log)" :key="index">{{ line }}</div>-->
<!--						</template>-->
<!--					</Column>-->
<!--				</DataTable>-->
				<div class="card">
					<DataTable :value="products" tableStyle="min-width: 50rem">
						<Column v-for="col of columns" :key="col.field" :field="col.field" :header="col.header"></Column>
					</DataTable>
				</div>
				
<!--				<DataTable :value="data" tableStyle="min-width: 50rem">-->
<!--					<Column v-for="(line, index) in splitLog" :key="index" :line="line"></Column>-->
<!--				</DataTable>-->
			
			</div>
		
		
		</div>
	
	</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import InputText from "primevue/inputtext";
import axios from "axios";
import Button from "primevue/button";
import Calendar from "primevue/calendar";
import MultiSelect  from 'primevue/multiselect';
import DataTable  from 'primevue/datatable';
import Column  from 'primevue/datatable';
import { ref, onMounted } from 'vue';


export default defineComponent({
	name: "HomeView",
	components: {
		InputText,
		Button,
		Calendar,
		MultiSelect,
		DataTable,
		Column,
	},
	data() {
		return {
			url: "",
			title: "",
			editedURL: "",
			editing: false,
			time: "",
			options: [
				{ name: 'New York', code: 'NY' },
				{ name: 'Rome', code: 'RM' },
				{ name: 'London', code: 'LDN' },
				{ name: 'Istanbul', code: 'IST' },
				{ name: 'Paris', code: 'PRS' }
			],
			selectedItems: [],
			data:[
				{
					log:
						'[13:36:53] Расчетное время: 9 мин[13:36:58] Открыть клапан откачки К1[13:36:58] Включить вакуумный насос[13:36:58] Закрыть клапан К5[13:36:58] Закрыть клапан дистилляции К2[13:36:58] Ожидание: 8 с[13:37:06] Заливка 2.2мл. в испаритель[13:37:06] Заданно 26.50602409638554 шагов[13:37:09] Заливка перекиси завершена[13:37:09] Открыть клапан дистилляции К2[13:42:09] Включить нагрев испарителя[13:42:09] Закрыть клапан дистилляции К2[13:42:09] конечное давление1.0960040758227925 торр[13:42:09] Выпаривание через К2[13:42:09] Выпаривание длилось5 мин[13:42:09] Откачка до 1 торр[13:42:15] Закрыть клапан откачки К1[13:43:09] Открыть клапан откачки К1[13:43:14] Аппаратное смещение 0 денсит. = -0.313683180809021[13:43:14] Закрыть клапан дистилляции К2',
					
				}
			],
			products: ref(),
			columns : [
				{ field: 'code', header: 'Code' },
				{ field: 'name', header: 'Name' },
				{ field: 'category', header: 'Category' },
				{ field: 'quantity', header: 'Quantity' }
			],
		};
	},
	computed: {
		formattedTime(): string {
			if(!this.time) return "";
			const date = new Date(this.time);
			const hours = date.getHours().toString().padStart(2, "0");
			const minutes = date.getMinutes().toString().padStart(2, "0");
			return `${hours}:${minutes}`;
		}
	},
	mounted() {
		this.getTitle();
	},
	
	methods: {
		splitLog(log:any) {
			return log.split(/(?<=])\s/);
		},
		
		startEditing() {
			this.editing = true;
			this.editedURL = this.url;
		},
		
		saveEdit() {
			this.url = this.editedURL;
			this.editing = false;
			this.getTitle();
		},
		
		cancelEdit() {
			this.editing = false;
		},
		
		getTitle() {
			axios.get(this.url).then((response) => {
				const parser = new DOMParser();
				const htmlDoc = parser.parseFromString(response.data, "text/html");
				this.title = htmlDoc.getElementsByTagName("title")[0].innerText;
			}).catch(() => {
				this.title = "";
			});
		}
	}
});
</script>
