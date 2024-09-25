<script>
  import { Plus, Trash, ImagePlus, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxIdName: '',
      fromTaxId: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxIdName: '',
      toTaxId: '',
      toContact: { mail: '', phone: '' }
    },
    items: [],
    taxPercent: '',
    discountPercent: '',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: ''
    },
    currency: 'USD'
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar', flag: '🇺🇸' },
    // ... other currencies
  ];

  function getCurrencySymbol(code) {
    const currency = currencyOptions.find((c) => c.code === code);
    return currency ? currency.symbol : '';
  }

  function handleImageChange(event) {
    const file = event.target.files[0];

    if (
      file &&
      (file.type === 'image/png' || file.type === 'image/jpeg' || file.type === 'image/webp')
    ) {
      const reader = new FileReader();
      reader.onload = () => {
        appState.company.logo = reader.result;
        save();
      };
      reader.readAsDataURL(file);
    } else {
      alert('Please select a valid image (.png, .jpg, or .webp)');
    }
  }

  function save() {
    localStorage.setItem('invoiceData', JSON.stringify(appState));
  }

  function addItem() {
    if (itemDesc != '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc,
          price: parseFloat(itemPrice).toFixed(2),
          quantity: parseFloat(itemQty).toFixed(2)
        }
      ];

      itemDesc = '';
      itemPrice = 1;
      itemQty = 1;

      save();
      document.getElementById('descBox').focus();
    }
  }

  function deleteItem(index) {
    appState.items = appState.items.filter((_, i) => i !== index);
    save();
  }

  function reset() {
    appState = {
      company: { name: 'Wayne Enterprises', logo: '' },
      invoice: {
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'Enter Tax Name',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Enter Tax Name',
        toTaxId: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'SEO Consultation', price: '5000', quantity: '1' },
        { desc: 'Social Media Strategy', price: '2000', quantity: '3' },
        { desc: 'Content Creation', price: '10000', quantity: '1' }
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: ''
      },
      currency: 'USD'
    };

    save();
  }

  onMount(() => {
    const savedData = localStorage.getItem('invoiceData');
    if (savedData) {
      appState = JSON.parse(savedData);
    } else {
      reset();
    }
  });

  let itemDesc = '';
  let itemPrice = 1;
  let itemQty = 1;

  $: subTotal = appState.items
    .reduce((total, item) => {
      return total + item.price * item.quantity;
    }, 0)
    .toFixed(2);

  $: discountAmount = ((subTotal * (+appState.discountPercent || 0)) / 100).toFixed(2);
  $: taxableAmount = (subTotal - discountAmount).toFixed(2);
  $: taxAmount = ((taxableAmount * (+appState.taxPercent || 0)) / 100).toFixed(2);
  $: totalDue = (parseFloat(taxableAmount) + parseFloat(taxAmount)).toFixed(2);

  // Toggles
  let isDiscountEnabled = true;
  let isTaxEnabled = true;

  // Auto-expand textarea function
  function autoExpand(field) {
    field.style.height = 'inherit';
    const computed = window.getComputedStyle(field);
    const height =
      parseInt(computed.getPropertyValue('border-top-width'), 10) +
      parseInt(computed.getPropertyValue('padding-top'), 10) +
      field.scrollHeight +
      parseInt(computed.getPropertyValue('padding-bottom'), 10) +
      parseInt(computed.getPropertyValue('border-bottom-width'), 10);

    field.style.height = height + 'px';
  }
</script>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-xl mx-auto px-4 sm:px-6 py-8 flex flex-col space-y-6 font-montserrat bg-white border-2 border-gray-600 rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Company & Invoice Details -->
  <div class="flex flex-col sm:flex-row justify-between items-start">
    <div class="flex flex-col gap-4 w-full sm:w-auto">
      <!-- Company Logo -->
      <div>
        {#if appState.company.logo}
          <div class="flex items-center space-x-4">
            <div class="flex flex-col border-2 border-gray-600 rounded-xl">
              <button
                class="p-2 hover:bg-gray-200 rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash />
              </button>
              <button
                class="p-2 hover:bg-gray-200 rounded"
                on:click={() => document.getElementById('imageInput').click()}
              >
                <ImagePlus />
                <input
                  id="imageInput"
                  type="file"
                  accept=".png,.jpg,.jpeg,.webp"
                  on:change={handleImageChange}
                  class="hidden"
                />
              </button>
            </div>
            <img
              src={appState.company.logo}
              alt="Company Logo"
              class="max-h-18 max-w-40 object-cover m-0"
            />
          </div>
        {:else}
          <button
            class="p-2 flex items-center gap-2 rounded-lg border-2 border-gray-600 cursor-pointer hover:bg-gray-100"
            on:click={() => document.getElementById('imageInput').click()}
          >
            <ImagePlus />
            <p class="text-sm text-gray-700">Click to select an image for logo</p>
            <input
              id="imageInput"
              type="file"
              accept=".png,.jpg,.jpeg,.webp"
              on:change={handleImageChange}
              class="hidden"
            />
          </button>
        {/if}
      </div>
      <!-- Company Name -->
      <input
        class="font-bold text-xl border-2 border-gray-600 p-3 rounded-lg focus:outline-none focus:border-gray-800 w-full"
        type="text"
        bind:value={appState.company.name}
      />
    </div>

    <!-- Invoice Details -->
    <div class="relative flex flex-col items-start sm:items-end gap-2 mt-4 sm:mt-0 w-full sm:w-auto">
      <h2 class="font-bold text-2xl text-green-700 text-left sm:text-right">
        Invoice :
        <input
          type="text"
          size="4"
          placeholder="#0001"
          maxlength="5"
          bind:value={appState.invoice.number}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800"
        />
      </h2>
      <div class="flex flex-row gap-4">
        <p class="flex items-center">
          <span class="mr-2">Created :</span>
          <input
            bind:value={appState.invoice.created}
            type="date"
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800"
          />
        </p>
        <p class="flex items-center">
          <span class="mr-2">Due :</span>
          <input
            bind:value={appState.invoice.due}
            type="date"
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800"
          />
        </p>
      </div>
      <select
        bind:value={appState.currency}
        class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
      >
        {#each currencyOptions as option}
          <option value={option.code}>
            {option.flag} {option.code} - {option.name}
          </option>
        {/each}
      </select>
    </div>
  </div>

  <hr class="border-gray-600 border-2" />

  <!-- Contact Details -->
  <div class="flex flex-col sm:flex-row justify-between gap-4">
    <!-- Sender Info -->
    <div class="flex flex-col gap-2 w-full">
      <h2 class="mb-2 font-bold">Sender Info</h2>
      <textarea
        placeholder="Enter sender's name and address"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
        on:input={(e) => autoExpand(e.target)}
        rows="1"
      ></textarea>
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <textarea
            bind:value={appState.invoice.fromTaxIdName}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Tax Name"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <textarea
            bind:value={appState.invoice.fromTaxId}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Tax ID"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <textarea
            bind:value={appState.invoice.fromContact.mail}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Email"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <textarea
            bind:value={appState.invoice.fromContact.phone}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Phone"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
      </div>
    </div>
    <!-- Recipient Info -->
    <div class="flex flex-col gap-2 w-full">
      <h2 class="mb-2 font-bold">Recipient Info</h2>
      <textarea
        placeholder="Enter recipient's name and address"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
        on:input={(e) => autoExpand(e.target)}
        rows="1"
      ></textarea>
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <textarea
            bind:value={appState.invoice.toTaxIdName}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Tax Name"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <textarea
            bind:value={appState.invoice.toTaxId}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Tax ID"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <textarea
            bind:value={appState.invoice.toContact.mail}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Email"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <textarea
            bind:value={appState.invoice.toContact.phone}
            class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800 w-full"
            placeholder="Enter Phone"
            on:input={(e) => autoExpand(e.target)}
            rows="1"
          ></textarea>
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form -->
  <form class="flex flex-col sm:flex-row justify-between gap-2">
    <div class="flex items-center">
      <button
        disabled={itemDesc == ''}
        on:click={() => addItem()}
        class="p-2 rounded-full bg-gray-200 hover:bg-gray-300 transition-all duration-100 ease-in-out print:hidden"
      >
        <Plus stroke="#557571" />
      </button>
    </div>

    <div class="flex flex-col sm:flex-row gap-2 w-full">
      <textarea
        id="descBox"
        bind:value={itemDesc}
        placeholder="Item name"
        class="border-2 border-gray-600 rounded-lg p-3 focus:outline-none focus:border-gray-800 grow w-full resize-none"
        rows="1"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
        on:input={(e) => autoExpand(e.target)}
      ></textarea>
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        class="border-2 border-gray-600 rounded-lg p-3 focus:outline-none focus:border-gray-800 w-full sm:w-32 text-right"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        class="border-2 border-gray-600 rounded-lg p-3 focus:outline-none focus:border-gray-800 w-full sm:w-32 text-right"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
      />
      <p class="border-2 border-gray-600 p-3 w-full sm:w-32 text-right bg-gray-100">
        {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
      </p>
    </div>
  </form>

  <!-- Item Descriptions -->
  <div class="flex flex-col mt-4 p-2 rounded-lg border-2 border-gray-600">
    <h2 class="mb-2 font-bold">Item Descriptions</h2>
    <div class="flex flex-row bg-gray-200 border-b-2 border-gray-600">
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/2">Item Name</p>
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/6 text-right">Price</p>
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/6 text-right">Qty</p>
      <p class="font-bold p-3 w-1/6 text-right">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row even:bg-blue-50 border-b-2 border-gray-600">
        <div class="flex items-center w-1/2">
          <button
            on:click={() => deleteItem(index)}
            class="p-3 hover:bg-gray-200 transition-all duration-100 ease-in-out rounded print:hidden"
          >
            <Trash />
          </button>
          <textarea
            class="p-3 grow resize-none bg-blue-100"
            bind:value={item.desc}
            rows="1"
            on:input={(e) => {
              autoExpand(e.target);
              save();
            }}
          ></textarea>
        </div>
        <input
          class="p-3 w-1/6 text-right bg-blue-100"
          type="number"
          min="0"
          step="0.01"
          bind:value={item.price}
          on:input={() => save()}
        />
        <input
          class="p-3 w-1/6 text-right bg-blue-100"
          type="number"
          min="0"
          step="1"
          bind:value={item.quantity}
          on:input={() => save()}
        />
        <p class="p-3 w-1/6 text-right bg-gray-100">
          {getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}
        </p>
      </div>
    {/each}

    <!-- Summary Rows -->
    <div class="flex flex-row items-center border-t-2 border-gray-600">
      <p class="p-3 w-5/6 text-right font-bold">Discount</p>
      <div class="flex items-center w-1/6">
        <input
          class="p-3 text-right bg-blue-100"
          type="number"
          bind:value={appState.discountPercent}
          disabled={!isDiscountEnabled}
          step="0.01"
          max="100"
          min="0"
          on:input={() => save()}
        />
        <span class="ml-1">%</span>
      </div>
    </div>
    <div class="flex flex-row items-center border-t-2 border-gray-600">
      <p class="p-3 w-5/6 text-right font-bold">Tax</p>
      <div class="flex items-center w-1/6">
        <input
          class="p-3 text-right bg-blue-100"
          type="number"
          bind:value={appState.taxPercent}
          disabled={!isTaxEnabled}
          step="0.01"
          max="100"
          min="0"
          on:input={() => save()}
        />
        <span class="ml-1">%</span>
      </div>
    </div>
    <div class="flex flex-row border-t-2 border-gray-600">
      <p class="p-3 w-5/6 text-right font-bold">Total Due</p>
      <p class="p-3 w-1/6 text-right bg-green-100 font-bold">
        {getCurrencySymbol(appState.currency)}{totalDue}
      </p>
    </div>
  </div>

  <!-- Payment Info -->
  <div class="mt-4">
    <h2 class="mb-2 font-bold">Payment Info</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <textarea
          bind:value={appState.payment.accountNumber}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <textarea
          bind:value={appState.payment.accountName}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-gray-800"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <!-- Additional Payment Fields -->
    </div>
  </div>

  <!-- Download Button -->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-gray-200 text-gray-800 font-semibold flex items-center gap-2 hover:bg-gray-300 transition-transform duration-150"
    >
      <Download class="w-5 h-5" />
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  .toggle-switch {
    width: 36px;
    height: 18px;
    background-color: #e2e2e2;
    border-radius: 20px;
    position: relative;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .toggle-switch:before {
    content: '';
    position: absolute;
    top: 1px;
    left: 1px;
    width: 16px;
    height: 16px;
    background-color: #8eaccd;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #8eaccd;
  }

  .toggle-switch.active:before {
    transform: translateX(18px);
    background-color: #333333;
  }

  .enabled {
    background-color: #ffffff;
  }

  .disabled {
    background-color: #f0f0f0;
  }

  /* Ensure text areas auto-expand */
  textarea {
    overflow: hidden;
    resize: none;
  }

  /* Adjust borders for better visibility */
  .border-2 {
    border-width: 2px;
  }

  /* Print styles */
  @media print {
    .toggle-switch {
      display: none;
    }
    button {
      display: none;
    }
    input,
    textarea {
      border: none;
    }
    .print\:hidden {
      display: none;
    }
  }
</style>
