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
    { code: 'EUR', symbol: '€', name: 'Euro', flag: '🇪🇺' },
    { code: 'GBP', symbol: '£', name: 'British Pound', flag: '🇬🇧' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen', flag: '🇯🇵' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar', flag: '🇦🇺' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar', flag: '🇨🇦' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc', flag: '🇨🇭' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan', flag: '🇨🇳' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee', flag: '🇮🇳' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble', flag: '🇷🇺' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real', flag: '🇧🇷' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand', flag: '🇿🇦' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso', flag: '🇲🇽' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar', flag: '🇳🇿' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar', flag: '🇸🇬' }
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
</script>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-4 sm:px-6 py-8 flex flex-col space-y-6 font-montserrat bg-white border border-gray-400 rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-col sm:flex-row justify-between items-start print:flex border-b-2 border-gray-400 pb-4">
    <div class="flex flex-col gap-4 w-full sm:w-auto">
      <div class="">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-col border border-gray-400 rounded-xl">
              <button
                class="p-2 hover:bg-gray-300 rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash />
              </button>
              <button class="p-2 hover:bg-gray-300 rounded" on:click={() => document.getElementById('imageInput').click()}>
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
            class="p-2 flex flex-row gap-2 rounded-lg border border-gray-400 cursor-pointer hover:bg-gray-200"
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
      <input
        class="font-bold text-xl border border-gray-400 p-3 rounded-lg focus:outline-none focus:border-gray-600 w-full"
        type="text"
        bind:value={appState.company.name}
        placeholder="Company Name"
      />
    </div>

    <div class="relative flex flex-col items-start sm:items-end gap-2 mt-4 sm:mt-0 w-full sm:w-auto">
      <h2 class="font-bold text-2xl text-gray-800 text-left sm:text-right print:text-left">
        Invoice :
        <input
          type="text"
          size="6"
          placeholder="#0001"
          maxlength="10"
          bind:value={appState.invoice.number}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
        />
      </h2>
      <div class="flex flex-col gap-1">
        <p class="flex items-center">
          <span class="mr-2">Created :</span>
          <input
            bind:value={appState.invoice.created}
            type="date"
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          />
        </p>
        <p class="flex items-center">
          <span class="mr-2">Due :</span>
          <input
            bind:value={appState.invoice.due}
            type="date"
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          />
        </p>
      </div>
      <select bind:value={appState.currency} class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full">
        {#each currencyOptions as option}
          <option value={option.code}>
            {option.flag} {option.code} - {option.name}
          </option>
        {/each}
      </select>
    </div>
  </div>

  <!-- From & To  ------------------------------------------------------>
  <div class="flex flex-col sm:flex-row justify-between gap-4 border-b-2 border-gray-400 pb-4">
    <div class="flex flex-col gap-2 w-full border p-4 rounded-lg">
      <h2 class="mb-2 font-bold">Sender Info</h2>
      <textarea
        style="resize: vertical; padding: 8px 12px;"
        placeholder="Enter sender's name and address"
        rows="3"
        maxlength="500"
        bind:value={appState.invoice.from}
        class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full break-words"
      ></textarea>
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
    <div class="flex flex-col gap-2 w-full border p-4 rounded-lg">
      <h2 class="mb-2 font-bold">Recipient Info</h2>
      <textarea
        style="resize: vertical; padding: 8px 12px;"
        placeholder="Enter recipient's name and address"
        rows="3"
        maxlength="500"
        bind:value={appState.invoice.to}
        class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full break-words"
      ></textarea>
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600 w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form --------------------------------------------------->
  <div class="border p-4 rounded-lg">
    <form class="flex flex-col sm:flex-row justify-between gap-2">
      <div class="flex items-center">
        <button
          disabled={itemDesc == ''}
          on:click={() => addItem()}
          class="p-2 rounded-full bg-gray-400 hover:bg-gray-500 transition-all duration-100 ease-in-out print:hidden"
        >
          <Plus stroke="#ffffff" />
        </button>
      </div>

      <div class="flex flex-col sm:flex-row gap-2 w-full">
        <input
          id="descBox"
          type="text"
          bind:value={itemDesc}
          placeholder="Item name"
          class="border border-gray-400 rounded-lg p-3 focus:outline-none focus:border-gray-600 grow w-full"
          on:keypress={(e) => e.key == 'Enter' && addItem()}
        />
        <input
          type="number"
          bind:value={itemPrice}
          placeholder="Unit Price"
          min="0"
          step="0.01"
          class="border border-gray-400 rounded-lg p-3 focus:outline-none focus:border-gray-600 w-full sm:w-32"
          on:keypress={(e) => e.key == 'Enter' && addItem()}
        />
        <input
          type="number"
          bind:value={itemQty}
          placeholder="Quantity"
          min="1"
          step="1"
          class="border border-gray-400 rounded-lg p-3 focus:outline-none focus:border-gray-600 w-full sm:w-32"
          on:keypress={(e) => e.key == 'Enter' && addItem()}
        />
        <p class="border border-gray-400 p-3 w-full sm:w-32 text-right">
          {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
        </p>
      </div>
    </form>
  </div>

  <!-- Table ----------------------------------------------------------->
  <div class="flex flex-col mt-4 border p-4 rounded-lg">
    <h2 class="mb-2 font-bold">Item Descriptions</h2>
    <div class="flex flex-row bg-gray-100 border border-gray-400">
      <p class="font-bold p-3 border-r border-gray-400 grow">Item name</p>
      <p class="font-bold p-3 border-r border-gray-400 w-32 hidden sm:block">Unit Price</p>
      <p class="font-bold p-3 border-r border-gray-400 w-24 hidden sm:block">Qty</p>
      <p class="font-bold p-3 w-32 text-right hidden sm:block">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-col sm:flex-row even:bg-gray-50 border border-gray-400">
        <div class="flex flex-row items-center">
          <button
            on:click={() => deleteItem(index)}
            class="p-3 hover:bg-gray-300 transition-all duration-100 ease-in-out rounded print:hidden"
          >
            <Trash />
          </button>
          <textarea
            class="p-3 border-r border-gray-400 grow break-words resize-none focus:outline-none focus:border-gray-600"
            bind:value={item.desc}
            on:input={() => save()}
            rows="1"
          ></textarea>
        </div>
        <div class="flex flex-row sm:flex-row w-full">
          <input
            class="p-3 border-r border-gray-400 w-full sm:w-32 focus:outline-none focus:border-gray-600"
            type="number"
            min="0"
            step="0.01"
            bind:value={item.price}
            on:input={() => save()}
          />
          <input
            class="p-3 border-r border-gray-400 w-full sm:w-24 focus:outline-none focus:border-gray-600"
            type="number"
            min="0"
            step="1"
            bind:value={item.quantity}
            on:input={() => save()}
          />
          <p class="p-3 w-full sm:w-32 text-right">
            {getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}
          </p>
        </div>
      </div>
    {/each}

    <div class="flex flex-row even:bg-gray-50 border-t border-gray-400 items-center">
      <p class="p-3 grow text-right font-bold">Discount %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isDiscountEnabled ? 'active' : ''}"
        on:click={() => (isDiscountEnabled = !isDiscountEnabled)}
      ></button>
      <input
        class="p-3 w-32 text-right {isDiscountEnabled ? 'enabled' : 'disabled'} focus:outline-none focus:border-gray-600"
        type="number"
        bind:value={appState.discountPercent}
        disabled={!isDiscountEnabled}
        step="0.01"
        max="100"
        min="0"
        on:input={() => save()}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 border-t border-gray-400 items-center">
      <p class="p-3 grow text-right font-bold">Tax %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isTaxEnabled ? 'active' : ''}"
        on:click={() => (isTaxEnabled = !isTaxEnabled)}
      ></button>
      <input
        class="p-3 w-32 text-right {isTaxEnabled ? 'enabled' : 'disabled'} focus:outline-none focus:border-gray-600"
        type="number"
        bind:value={appState.taxPercent}
        disabled={!isTaxEnabled}
        step="0.01"
        max="100"
        min="0"
        on:input={() => save()}
      />
    </div>
    <div class="flex flex-row even:bg-gray-50 border-t border-gray-400">
      <p class="p-3 grow text-right font-bold">Total Due</p>
      <p class="p-3 w-32 text-right font-bold">
        {getCurrencySymbol(appState.currency)}{totalDue}
      </p>
    </div>
  </div>

  <!-- Payment Info Section Redesigned --------------------------------->
  <div class="mt-4 border p-4 rounded-lg">
    <h2 class="mb-2 font-bold">Payment Info</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <input
          type="text"
          maxlength="20"
          bind:value={appState.payment.accountNumber}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          placeholder="Enter Account Number"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.accountName}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          placeholder="Enter Account Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.bank}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          placeholder="Enter Bank Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.ifsc}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          placeholder="Enter IFSC Code"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.swiftCode}
          class="border border-gray-400 p-2 rounded-lg focus:outline-none focus:border-gray-600"
          placeholder="Enter SWIFT Code"
        />
      </div>
    </div>
  </div>

  <!-- Download Button -------------------------------------------------->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-gray-400 text-white font-semibold flex items-center gap-2 hover:bg-gray-500 transition-transform duration-150"
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
    background-color: #ffffff;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #4a90e2;
  }

  .toggle-switch.active:before {
    transform: translateX(18px);
    background-color: #ffffff;
  }

  .enabled {
    background-color: #ffffff;
    color: #000000;
  }

  .disabled {
    background-color: #f0f0f0;
    color: #a0a0a0;
  }

  /* Ensure text wraps inside boxes */
  textarea,
  p,
  input {
    word-wrap: break-word;
    overflow-wrap: break-word;
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
    .print:hidden {
      display: none;
    }
    body {
      -webkit-print-color-adjust: exact;
    }
  }

  /* Adjust border visibility */
  .border {
    border-width: 1.5px;
  }

  /* Increase spacing between sections */
  .border-b-2 {
    margin-bottom: 1rem;
  }
</style>
