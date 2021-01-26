<template>
  <panel-item :field="field">
    <template slot="value" v-if="field.value && field.value.length">
      <span v-if="field.pivots"> 
      	<div  
		  	style="display: flex; align-items: center; margin-bottom: 5px"
      		class="no-underline font-bold dim text-primary cursor-pointer"
        	v-for="(resource, index) in field.value"
        	:key="resource.id"
        	@click="displayPivots(resource)"
        	:title="__(':resource Details', { resource: resource.text })"
        >
		  <table class="container-pivot" style="table-layout: auto">
		  	<tr v-if="index == 0">
				<td></td>
				<td v-for="pivotvalue in pivotsDisplay(resourceName, field.value, resource.pivotId, 'label')" :key="pivotvalue.label">{{pivotvalue['label']}}</td>
			</tr>
			<tr>
				<td>{{resource.text}}</td>
				<td v-for="pivotvalue in pivotsDisplay(resourceName, field.value, resource.pivotId, 'value')" :key="pivotvalue.label">{{pivotvalue['value']}}</td>
			</tr>  
		  </table>	
        </div>
      </span> 
      <span v-else>
        <router-link 
          v-for="(resource, index) in field.value"
          :key="index"
          :to="{
            name: 'detail',
            params: {
              resourceName: field.resourceName,
              resourceId: resource.id,
            },
          }"
          class="no-underline font-bold dim text-primary"
          :title="__('View')"
        > 
          {{ resource.text }} {{ field.value.length - index - 1 ? ' , ' : ''  }}
        </router-link>
      </span>
       <modal v-if="field.pivots && display" role="dialog" @modal-close="handleClose">  
            <loading-view :loading="loading"> 
							<card :class="'w-action-fields'"> 
			          <form-heading-field 
			            :field="{
			              asHtml: true,
			              value: display.text  
			            }" 
			            class="mb-6 pt-6"
			          />
 
	    					<div class="mb-6 py-3 px-6">
	                <!-- Pivot Fields --> 
	                <component
	                	v-for="(field, index) in fields"
	                	:key="index"
	                  :is="'detail-' + field.component"
	                  :resource-name="resourceName"
	                  :field="field" 
	                  :via-resource="viaResource"
	                  :via-resource-id="viaResourceId"
	                  :via-relationship="viaRelationship"
	                /> 
	              </div>

	              <div class="px-6 py-3 flex">
	                <div class="flex items-center ml-auto">
			            	<button  
                    	class="btn btn-link dim cursor-pointer text-80 ml-auto mr-6"
			            		type="button" 
			            		@click.prevent="handleClose">{{ 
				            	__("Close") 
				            }}</button>
			            	<router-link 
							        :to="{
							          name: 'detail',
							          params: {
							            resourceName: field.resourceName,
							            resourceId: display.id,
							          },
							        }"
							        class="no-underline font-bold dim text-primary"
            					:title="__('View')"
							      > 
            				{{ __(':resource Details', { resource: display.text }) }}
							      </router-link>
							    </div>
							  </div>  
            	</card>
            </loading-view>
       </modal>
    </template>
    <p v-else slot="value">&mdash;</p>
  </panel-item>
</template>

<script>
export default {
    props: ['resource', 'resourceName', 'resourceId', 'field'],
	mounted (){
	},
    data() {
    	return {
    		display: false,
    		fields: [],
    	}
    },
	computed: {
		
	},
    methods: {
		pivotsDisplay(resourceName, value, pivotId){
			console.log(resourceName, value);
			let v = value.find((e) => e.pivotId == pivotId);
			if(resourceName == 'menus' && v){
				return v.pivot_info;
			}
			return [];
		},
    	displayPivots(resource) {
    		this.display = resource;
    		this.loading = false;

    		this.getPivotFields()  
    	}, 

      /**
       * Get all of the available fields for an attachment.
       */
      async getPivotFields(resource) {   
        await Nova.request()
          .get(
            `/nova-vendor/many-to-many/${this.resourceName}/pivot-fields/${this.field.resourceName}`,
            {
              params: { 
                resourceId: this.resourceId,
                relatedId: this.display.id,   
                pivotId: this.display.pivotId,   
              },
            }
          ) 
          .then(({ data }) => { 
            this.fields = data

            _.each(this.fields, field => {
              field.fill = () => ''   
            })
          }) 
      }, 

	    /**
	     * Resolve the component name.
	     */
	    resolveComponentName(field) {
	      return field.prefixComponent
	        ? 'detail-' + field.component
	        : field.component
	    },

    	handleClose() {
    		this.display = null

        this.$emit('close')
    	},
    },

    computed: {
    },
}
</script>

<style>
	.container-pivot {
  table-layout: fixed ;
  width: 100% ;
}
.container-pivot td {
  width: 25% ;
  padding: 10px;
}
</style>
