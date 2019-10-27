<link rel="shortcut icon" type="image/png" href="assets/images/favicon.ico">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div dir="ltr" style="text-align: left;" trbidi="on">
<div class="separator" style="clear: both; text-align: center;">
<a href="https://1.bp.blogspot.com/-RrJFVJiBuI4/XWzSp5lh9UI/AAAAAAAAAZA/XHecVbKiygkitkJ1LAjru0_FZn_eGKdZgCLcBGAs/s1600/Dependent-Picklist1-compressor.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img alt="Picklist and Dependent Picklist in Salesforce LWC and AURA" border="0" data-original-height="410" data-original-width="640" src="https://1.bp.blogspot.com/-RrJFVJiBuI4/XWzSp5lh9UI/AAAAAAAAAZA/XHecVbKiygkitkJ1LAjru0_FZn_eGKdZgCLcBGAs/s1600/Dependent-Picklist1-compressor.png" title="Picklist and Dependent Picklist in Salesforce LWC and AURA" /></a></div>
<div class="separator" style="clear: both; text-align: center;">
</div>


<div style="text-align: justify;">
<span style="font-size: large;">We will utilize base components that Salesforce has provided to build custom dynamic Picklist and Dependent Picklist.</span></div>



<!--more-->
<h2 style="text-align: left;">
Base component that we will be using:</h2>
<pre class="lang-markup" width="100%"><code>
&lt;lightning-combobox
            name="progress"
            label="Status"
            value={value}
            placeholder="Select Progress"
            options={options}
            onchange={handleChange}&gt;
&lt;/lightning-combobox&gt;

</code></pre>


</div>


</div>
<div style="text-align: justify;">
<div style="text-align: left;">
<span style="font-size: large;">We will be using the above base component to build custom dynamic picklist and dependent picklist components, which can be re-used anywhere just by passing some parameters. We will see how we can use that custom component in Aura, which covers AURA inter-operability as well.</span></div>
</div>
<span style="font-size: large;"><span style="font-size: large;">
</span> </span>
<h2 style="text-align: left;">
<span style="font-size: large;"> Main Logic for dependent picklist:</span></h2>
<div style="text-align: justify;">
<div style="text-align: left;">
<span style="font-size: large;">We will reset the dependent picklist value on onchange handler of picklist, by firing pubsub events.</span></div>
</div>

<h2 style="text-align: left;">
Before you even see the code experience LIVE component here:</h2>

<div class="">
     {% include pick.html %}
</div>
    

<h2 style="text-align: left;">
Picklist.html</h2>
<pre class="lang-markup" width="100%"><code>
&lt;template&gt;
    &lt;lightning-combobox
        id="pickList"
        name="progress"
        label={label}
        value={value}
        variant={variant}
        placeholder="Select"
        options={options}
        onchange={handleChange}&gt;
    &lt;/lightning-combobox&gt;
&lt;/template&gt;
</code></pre>
</div>


<h2 style="text-align: left;">
Picklist.js</h2>
<div>
{% highlight javascript %}
/* eslint-disable no-console */
import { LightningElement, track, wire, api } from 'lwc';
import { getPicklistValuesByRecordType } from 'lightning/uiObjectInfoApi';
import { getObjectInfo } from 'lightning/uiObjectInfoApi';
import { CurrentPageReference } from 'lightning/navigation';
import { fireEvent } from 'c/pubsub';

export default class Picklist2 extends LightningElement {
    
    @wire(CurrentPageReference) pageRef;

    @api objectApiName;
    @api pickListfieldApiName;
    @api label;
    @api variant;

    /*only for lwc for mapping values in list and 
    also for mapping this with dependent picklist(give unique = record Id while using in dependent picklist)*/
    @api uniqueKey;

    @track value;
    recordTypeIdValue;

    @track options = [
        { label: 'Default 1', value: 'Default1' },
        { label: 'Default 2', value: 'Default2' },
        { label: '--None--', value: "" }
    ];   
    
    @api 
    get recordTypeId() {
        console.log("getter defaultRectype", this.recordTypeIdValue);
        return this.recordTypeIdValue;
    }
    set recordTypeId(value) {
        this.recordTypeIdValue = value;
        console.log("setter defaultRectype", this.recordTypeIdValue);
    }


    @api 
    get selectedValue() {
        console.log("getter", this.value);
        return this.value;
    }
    set selectedValue(val) {
        console.log("setter", val);
        if (val === '' || val === undefined || val === null)
            this.value = { label: '--None--', value: "" }.value;
        else
            this.value = val;
    }
         

    @wire(getObjectInfo, { objectApiName: '$objectApiName' })
    getRecordTypeId({ error, data }) {
        if (data) {
            this.record = data;
            this.error = undefined;
            if(this.recordTypeId === undefined){
                this.recordTypeId = this.record.defaultRecordTypeId;
            }
            console.log("Default Record Type Id", JSON.stringify(this.record.defaultRecordTypeId));
        } else if (error) {
            this.error = error;
            this.record = undefined;
            console.log("this.error",this.error);
        }
    }
                     
    @wire(getPicklistValuesByRecordType, { recordTypeId: '$recordTypeId', objectApiName: '$objectApiName' })
    wiredOptions({ error, data }) {
        if (data) {
            this.record = data;
            this.error = undefined;
            /*
            console.log("this.record picklist raw data", JSON.stringify(this.record));
            console.log("this.record.picklistFieldValues", JSON.stringify(this.record.picklistFieldValues));
            */
            if(this.record.picklistFieldValues[this.pickListfieldApiName] !== undefined) {

                let tempOptions = [{ label: '--None--', value: "" }];
                let temp2Options = this.record.picklistFieldValues[this.pickListfieldApiName].values;
                temp2Options.forEach(opt =&gt; tempOptions.push(opt));

                this.options = tempOptions;
            }
            console.log("this.options pick", JSON.stringify(this.options));
            if(this.selectedValue === '' || this.selectedValue === undefined || this.selectedValue === null) {
                this.value = { label: '--None--', value: "" }.value;
            } else {
                this.value = this.options.find(listItem =&gt; listItem.value === this.selectedValue).value;
            }
        } else if (error) {
            this.error = error;
            this.record = undefined;
            console.log("this.error",this.error);
        }
    }


    handleChange(event) {
        let tempValue = event.target.value;
        console.log("event.target.value",event.target.value);
        console.log("this.value",tempValue);
        let selectedValue = tempValue;
        let key = this.uniqueKey;

        //Firing change event for aura container to handle
        //For Self
        const pickValueChangeEvent = new CustomEvent('picklistchange', {
            detail: { selectedValue, key },
        });
        this.dispatchEvent(pickValueChangeEvent);

        //For dependent picklist
        let eventValues = {selValue : selectedValue, uniqueFieldKey: `${this.pickListfieldApiName}${this.uniqueKey}`};
        console.log("eventValues",JSON.stringify(eventValues));
        console.log("eventValues",eventValues);
        //Fire Pub/Sub Event, So that every other comp in the page knows the change
        fireEvent(this.pageRef, 'controllingValue', eventValues);
    }

}
{% endhighlight %}
</div>


<h2 style="text-align: left;">
Picklist.js-meta.xml</h2>
<pre class="lang-markup" width="100%"><code>
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata" fqn="picklist2"&gt;
    &lt;apiVersion&gt;46.0&lt;/apiVersion&gt;
    &lt;isExposed&gt;true&lt;/isExposed&gt;
    &lt;targets&gt;
        &lt;target&gt;lightning__AppPage&lt;/target&gt;
        &lt;target&gt;lightning__RecordPage&lt;/target&gt;
        &lt;target&gt;lightning__HomePage&lt;/target&gt;
    &lt;/targets&gt;   
&lt;/LightningComponentBundle&gt;
</code></pre>


<span style="font-size: large;">Here we are using UI API to get record type id and picklist values based on that. UI API is helping us solve most of the problem, Otherwise previously should have to make an API call to get these results.</span>
<span style="font-size: large;">This is one more credit we can give to LWC for making our lives easier.</span>
<span style="font-size: large;">
</span> <span style="font-size: large;">Above code is all about picklist what about dependent picklist ?? Hang on here comes it.</span>
<span style="font-size: large;">
</span> <span style="font-size: large;">I have tried all different means to remove parent child hierarchy between picklist and dependent picklist, there is no direct relation, but has a bit of relation that is because of events. Dependent picklist depends on the event fired by picklist. When onchange event occurs in picklist it fires a pubsub event to notify dependent picklist that its value has changed, there by depdendent picklist changes its value to none. I have tries all different approaches to remove this relation but the final outcome wasn't stable. That is the final outcome behaved differently in AURA and LWC, so had to to be content with this relation. Please leave your suggestions below to make this component even better. You can subscribe to the to my newsletter just by pressing on the '+' plus button to bottom right corner, it is also a lightning web component which is inturn integrated to mailchimp which is a e-mail service.</span>
<span style="font-size: large;">
</span> <span style="font-size: large;">Here comes dependent picklist code.</span>

<h2 style="text-align: left;">
dependentPickList.html:</h2>
<div>
<pre class="lang-markup" width="100%"><code>
&lt;template&gt;
    &lt;lightning-combobox id="dependentPickList"
                        name="Hello"
                        label={label}
                        value={value}
                        placeholder="--None--"
                        variant={variant}
                        options={options}
                        onchange={handleChange}&gt;
    &lt;/lightning-combobox&gt;
&lt;/template&gt;
</code></pre>
</div>
</div>


<h2 style="text-align: left;">
dependentPickList.js:</h2>
<pre class="lang-javascript" width="100%"><code>
/* eslint-disable no-console */
import { LightningElement, track, wire, api } from 'lwc';
import { getPicklistValuesByRecordType } from 'lightning/uiObjectInfoApi';
import { getObjectInfo } from 'lightning/uiObjectInfoApi';
import { CurrentPageReference } from 'lightning/navigation';
import { registerListener, unregisterAllListeners } from 'c/pubsub';
import { NavigationMixin } from 'lightning/navigation';
import { fireEvent } from 'c/pubsub';

export default class DependentPickList4 extends NavigationMixin(LightningElement) {

    @wire(CurrentPageReference) pageRef;

    @api objectApiName; 
    @api pickListfieldApiName; //this field api name
    @api controllingFieldApi; //parent field api name that is controlling field api name
    //@api controllingFieldValue; //parent field value
    @api label;
    @api variant;

    recordTypeIdValue;
    controllingFieldVal;
    previousValue;
    prevControllingFieldVal;
    @track value;

    /*only for lwc for mapping values in list and 
    also for mapping this dependent picklist with another dependent picklist(give unique = record Id while using in dependent picklist)*/
    @api uniqueKey;

    connectedCallback() {
        console.log("In connected callback depndent");
        registerListener('controllingValue', this.handelControllingValue, this);
    }

    @api
    get controllingFieldValue() {
        return this.controllingFieldVal;
    }
    set controllingFieldValue(value) {
        this.controllingFieldVal = value;
        this.reinitiatemap();
    }


    disconnectedCallback() {
        console.log("In disconnected callback depndent");
        unregisterAllListeners(this);
    }


    @api 
    get recordTypeId() {
        console.log("getter defaultRectype", this.recordTypeIdValue);
        return this.recordTypeIdValue;
    }
    set recordTypeId(value) {
        this.recordTypeIdValue = value;
        console.log("setter defaultRectype", this.recordTypeIdValue);
    }

    @api 
    get selectedValue() {
        console.log("getter dependent", this.value);
        return this.value;
    }
    set selectedValue(val) {
        console.log("setter dependent", val);
        this.previousValue = this.value;

        if (val === '' || val === undefined || val === null){
            this.value = { label: '--None--', value: "" }.value;
        }
        else
            this.value = val;
    }

    handelControllingValue(valuesObj) {
        console.log("In handelControllingValue", JSON.stringify(valuesObj));
        if (`${this.controllingFieldApi}${this.uniqueKey}` === valuesObj.uniqueFieldKey) {
            if (valuesObj.selValue === '' || valuesObj.selValue === null || valuesObj.selValue === undefined) {
                this.selectedValue = '';
                this.options = [{ label: '--None--', value: "" }];
            } else {
                this.selectedValue = '';
                if (this.myMap !== null &amp;&amp; this.myMap !== undefined) {

                    let tempOptions = [{ label: '--None--', value: "" }];
                        console.log("valuesObj.selValue", valuesObj.selValue);
                        if(this.myMap.get(valuesObj.selValue)) {
                            this.myMap.get(valuesObj.selValue).forEach(opt =&gt; tempOptions.push(opt));
                        }

                    this.options = tempOptions;
                }
            }

            let selectedValue = '';
            let key = this.uniqueKey;
            //Firing change event for aura container to handle
            //For Self
            const pickValueChangeEvent = new CustomEvent('picklistchange', {
                detail: { selectedValue, key },
            });
            this.dispatchEvent(pickValueChangeEvent);
            
            //For dependent picklist
            let eventValues = { selValue: '', uniqueFieldKey: `${this.pickListfieldApiName}${this.uniqueKey}` };
            //Fire Pub/Sub Event, So that every other comp in the page knows the change
            fireEvent(this.pageRef, 'controllingValue', eventValues);
        }
    }

    
    @track options = [
                      {label : 'Default 1', value : 'Default1'},
                      {label : 'Default 2', value : 'Default2'},
                      {label : '--None--', value : ''}
                     ];

    @track myMap = undefined;

    @wire(getObjectInfo, { objectApiName: '$objectApiName' })
    getRecordTypeId({ error, data }) {
        if (data) {
            this.record = data;
            this.error = undefined;
            if(this.recordTypeId === undefined){
                this.recordTypeId = this.record.defaultRecordTypeId;
            }
            console.log("Default Record Type Id", this.record.defaultRecordTypeId);
        } else if (error) {
            this.error = error;
            this.record = undefined;
            console.log("this.error",this.error);
        }
    }

    @wire(getPicklistValuesByRecordType, { recordTypeId: '$recordTypeId', objectApiName: '$objectApiName' })
    wiredOptions({ error, data }) {
        if (data) {
            this.record = data;
            this.error = undefined;

           let pickMap = new Map();

                if(this.record.picklistFieldValues[this.pickListfieldApiName] !== undefined) {
                    if(this.record.picklistFieldValues[this.pickListfieldApiName].controllerValues !== undefined) {

                        const controllerValues = this.record.picklistFieldValues[this.pickListfieldApiName].controllerValues;

                        Object.entries(controllerValues).forEach(([key, value]) =&gt;  {
                            const picValues = this.record.picklistFieldValues[this.pickListfieldApiName].values;
                            picValues.forEach(pickValue =&gt; {
                                if(pickValue.validFor.includes(value)) {
                                    if(pickMap.has(key)){
                                        let temp = pickMap.get(key);
                                        temp.push(pickValue);
                                        pickMap.set(key, temp);
                                    }else {
                                        let temp2 = [];
                                        temp2.push(pickValue);
                                        pickMap.set(key, temp2);
                                        console.log("In inner else", temp2);
                                    }
                                }
                                    
                            });
                        });
                        console.log("MAP",pickMap);
                        this.myMap = pickMap;
                        console.log("In Wire", this.myMap);
                        console.log("etter controllingFieldValue",this.controllingFieldValue);
                        //Checking if selected and controlling values exist, and populating values accordingly
                        if (this.selectedValue) {
                            this.value = this.selectedValue;
                            if (!this.controllingFieldValue) {
                                this.options = [{ label: this.selectedValue, value: this.selectedValue }];
                                return;
                            }
                        }
                        else if (!this.controllingFieldValue) {
                            this.options = [{ label: '--None--', value: '' }];
                            this.value = this.options[0].value;
                            return;
                        }
                        else if(!this.selectedValue) {
                            
                            this.value = { label: '--None--', value: '' }.value;
                        }

                        this.initiateMap(pickMap);
                        console.log("Initial selectedValue ", this.selectedValue);

                    }else {
                        console.log("Error: This field is not a dependent picklist!");
                    }
                }else {
                    console.log("Error in fetching picklist values! Invalid picklist field");
                }
            console.log("***Initial Options*** ", JSON.stringify(this.options));
        } else if (error) {
            this.error = error;
            this.record = undefined;
            console.log("this.error",this.error);
        }
    }

    handleChange(event) {
        console.log("etter selected val in handle change", this.selectedValue);
        this.value = event.target.value;
        console.log("event.target.value",event.target.value);
        console.log("this.value",this.value);
        let selectedValue = this.value;
        let key = this.uniqueKey;
        //Firing change event for aura container to handle
        //For Self
        const pickValueChangeEvent = new CustomEvent('picklistchange', {
            detail: { selectedValue, key },
        });
        this.dispatchEvent(pickValueChangeEvent);

        //For dependent picklist
        let eventValues = {selValue : selectedValue, uniqueFieldKey: `${this.pickListfieldApiName}${this.uniqueKey}`};
        //Fire Pub/Sub Event, So that every other comp in the page knows the change
        fireEvent(this.pageRef, 'controllingValue', eventValues);
    }
    
    initiateMap(thisMap) {
        this.myMap = thisMap;

        if (thisMap !== null &amp;&amp; thisMap !== undefined) {
            let tempOptions = [{ label: '--None--', value: "" }];
            if (this.controllingFieldValue !== null &amp;&amp; this.controllingFieldValue !== undefined &amp;&amp; this.controllingFieldValue !== '') {
                console.log("this.controllingFieldValue", this.controllingFieldValue);
                if(this.myMap.get(this.controllingFieldValue)) {
                    this.myMap.get(this.controllingFieldValue).forEach(opt =&gt; tempOptions.push(opt));
                }
            }
            this.options = tempOptions;
        }
        console.log("***Final Options dependent Picklist*** ", JSON.stringify(this.options));
    }
    
    
    reinitiatemap() {
        if (this.myMap !== null &amp;&amp; this.myMap !== undefined){
            this.initiateMap(this.myMap);
            console.log("this.myMap length", this.myMap.length);
        }
    }

}
</code></pre>


<h2 style="text-align: left;">
dependentPickList.js-meta.xml:</h2>
<pre class="lang-markup" width="100%"><code>
&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata" fqn="dependentPickList4"&gt;
    &lt;apiVersion&gt;46.0&lt;/apiVersion&gt;
    &lt;isExposed&gt;true&lt;/isExposed&gt;
    &lt;targets&gt;
        &lt;target&gt;lightning__AppPage&lt;/target&gt;
        &lt;target&gt;lightning__RecordPage&lt;/target&gt;
        &lt;target&gt;lightning__HomePage&lt;/target&gt;
    &lt;/targets&gt;   
&lt;/LightningComponentBundle&gt;
</code></pre>

<span style="color: #45818e; font-size: large;">
</span> 
<h2 style="text-align: left;">
Usage:</h2>
Picklist:
<pre class="language-markup" width="100%"><code>
&lt;c-picklist2
     unique-key={account.Id} 
     object-api-name="Account" 
     record-type-id="0127F000000kyxEQAQ" 
     selected-value={account.Master_Picklist__c}  
     pick-listfield-api-name="Master_Picklist__c"
     onpicklistchange={handleMasterPicklistChange}&gt;
&lt;/c-picklist2&gt;
</code></pre>
</div>


Dependent picklist:
<pre class="language-markup" width="100%"><code>
&lt;c-dependent-pick-list4
      unique-key={account.Id}
      object-api-name="Account"
      record-type-id="0127F000000kyxEQAQ" 
      pick-listfield-api-name="Controlling_Picklist__c"
      controlling-field-value={account.Master_Picklist__c}
      controlling-field-api="Master_Picklist__c"
      selected-value={account.Controlling_Picklist__c}
      onpicklistchange={handlePicklistChange}&gt;
&lt;/c-dependent-pick-list4&gt;
</code></pre>


<span style="color: #45818e; font-size: large;">
</span> <span style="color: #45818e; font-size: large;">Hey guys, If you find this post interesting and helpful don't forget to write your feedback down below in comments section. I am a social guy, you can find me in the apps mentioned below. Don't forget to share this with other Salesforce folks. Don't miss mentioning about Live components available in this site.</span>

<h2 style="text-align: left;">
Github Links:</h2>

<span style="font-size: large;">Picklist:</span>
<a href="https://github.com/kishoreBandanadam/lwc/tree/master/force-app/main/default/lwc/picklist2">https://github.com/kishoreBandanadam/lwc/tree/master/force-app/main/default/lwc/picklist2</a>

<span style="font-size: large;">DependentPicklist:</span>
<a href="https://github.com/kishoreBandanadam/lwc/tree/master/force-app/main/default/lwc/dependentPickList4">https://github.com/kishoreBandanadam/lwc/tree/master/force-app/main/default/lwc/dependentPickList4</a></div>
