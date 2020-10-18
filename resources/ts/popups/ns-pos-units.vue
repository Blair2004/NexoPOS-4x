<template>
    <div class="bg-white w-2/3-screen lg:w-1/3-screen overflow-hidden flex flex-col">
        <div id="header" class="h-16 flex justify-center items-center flex-shrink-0">
            <h3 class="font-bold text-gray-700">Choose Selling Unit</h3>
        </div>
        <div v-if="unitsQuantities.length > 0" class="grid grid-flow-row grid-cols-2 overflow-y-auto">
            <div @click="selectUnit( unitQuantity )" :key="unitQuantity.id" v-for="unitQuantity of unitsQuantities" class="hover:bg-gray-200 cursor-pointer border flex-shrink-0 border-gray-200 flex flex-col items-center justify-center">
                <div class="h-40 w-full flex items-center justify-center overflow-hidden">
                    <img v-if="unitQuantity.preview_url" :src="unitQuantity.preview_url" class="object-cover h-full" :alt="unitQuantity.unit.name">
                    <div class="h-40 flex items-center justify-center" v-if="! unitQuantity.preview_url">
                        <i class="las la-image text-gray-600 text-6xl"></i>
                    </div>
                </div>
                <div class="h-0 w-full">
                    <div class="relative w-full flex items-center justify-center -top-10 h-20 py-2" style="background:rgb(255 255 255 / 73%)">
                        <h3 class="text-sm font-bold text-gray-700 py-2 text-center">{{ unitQuantity.unit.name }} ({{ unitQuantity .quantity }})</h3>
                    </div>
                </div>
            </div>
        </div>
        <div class="h-56 flex items-center justify-center" v-if="unitsQuantities.length === 0">
            <ns-spinner></ns-spinner>
        </div>
    </div>
</template>
<script>
import { nsHttpClient, nsSnackBar } from '@/bootstrap';
export default {
    data() {
        return {
            unitsQuantities: []
        }
    },
    mounted() {
        this.$popup.event.subscribe( action => {
            if ( action.event === 'click-overlay' ) {
                /**
                 * as this runs under a Promise
                 * we need to make sure that
                 * it resolve false using the "resolve" function
                 * provided as $popupParams.
                 * Here we resolve "false" as the user has broken the Promise
                 */
                this.$popupParams.reject( false );

                /**
                 * we can safely close the popup.
                 */
                this.$popup.close();
            }
        });

        this.loadUnits();
    },
    methods: {
        loadUnits() {
            nsHttpClient.get( `/api/nexopos/v4/products/${this.$popupParams.product.$original().id}/units/quantities` )
                .subscribe( result => {
                    
                    if ( result.length === 0 ) {
                        this.$popup.close();
                        return nsSnackBar.error( 'This product doesn\'t has any unit defined for selling.' ).subscribe();
                    }

                    this.unitsQuantities  =   result;
                })
        },
        /**
         * we'll resolve a value that
         * will be added to the object
         * built at the end
         * @param Unit
         */
        selectUnit( unitQuantity ) {
            this.$popupParams.resolve({
                unit_quantity_id    :   unitQuantity.id,
                unit_name           :   unitQuantity.unit.name,
                $quantities         :   () => unitQuantity
            });
            this.$popup.close();
            // this.types.forEach( type => type.selected = false );
            // type.selected   =   true;
            // POS.order.types.next( this.types );
        }
    }
}
</script>