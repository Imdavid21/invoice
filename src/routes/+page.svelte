<script>
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      title: 'Invoice',
      number: '#0001',
      created: '',
      due: '',
      from: '',
      fromTaxDetails: '',
      fromContact: { mail: '', phone: '' },
      to: '',
      toTaxDetails: '',
      toContact: { mail: '', phone: '' }
    },
    items: [{ desc: '', price: '', quantity: '' }],
    taxPercent: '0',
    discountPercent: '0',
    note: '',
    payment: {
      accountNumber: '',
      accountName: '',
      bank: '',
      ifsc: '',
      swiftCode: '',
      note: ''
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
    if (itemDesc.trim() !== '' && itemPrice > 0 && itemQty > 0) {
      appState.items = [
        ...appState.items,
        {
          desc: itemDesc,
          price: parseFloat(itemPrice),
          quantity: parseFloat(itemQty)
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
        title: 'Invoice',
        number: '#0001',
        created: '',
        due: '',
        from: '',
        fromTaxDetails: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxDetails: '',
        toContact: { mail: '', phone: '' }
      },
      items: [
        { desc: 'SEO Consultation', price: '5000', quantity: '1' },
        { desc: 'Social Media Strategy', price: '2000', quantity: '3' },
        { desc: 'Content Creation', price: '10000', quantity: '1' }
      ],
      taxPercent: '0',
      discountPercent: '0',
      note: '',
      payment: {
        accountNumber: '',
        accountName: '',
        bank: '',
        ifsc: '',
        swiftCode: '',
        note: ''
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

  function formatNumber(num) {
    return num % 1 === 0 ? num.toString() : num.toFixed(2);
  }

  $: subTotal = appState.items
    .reduce((total, item) => {
      return total + parseFloat(item.price) * parseFloat(item.quantity);
    }, 0);

  $: if (!isDiscountEnabled) {
    appState.discountPercent = '0';
  }

  $: discountAmount = isDiscountEnabled
    ? ((subTotal * (+appState.discountPercent || 0)) / 100)
    : 0;

  $: taxableAmount = subTotal - discountAmount;

  $: if (!isTaxEnabled) {
    appState.taxPercent = '0';
  }

  $: taxAmount = isTaxEnabled
    ? ((taxableAmount * (+appState.taxPercent || 0)) / 100)
    : 0;

  $: totalDue = taxableAmount + taxAmount;

  // Toggles
  let isDiscountEnabled = true;
  let isTaxEnabled = true;
</script>

<svelte:head>
  <title>{appState.invoice.title} - {appState.company.name}</title>
  <meta
    name="description"
    content="Create and download invoices instantly, with zero signups or tracking."
  />
  <link
    href="https://fonts.googleapis.com/css2?family=DotGothic16&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-4 sm:px-6 py-8 flex flex-col space-y-6 invoice-container print:shadow-none print:bg-white print:border-none"
>
  <!-- INVOICE Title and Number at Top Center -->
  <div class="flex flex-col sm:flex-row justify-between items-center mb-6">
    <!-- Company Logo and Name -->
    <div class="flex items-center gap-4">
      {#if appState.company.logo}
        <img
          src={appState.company.logo}
          alt="Company Logo"
          class="h-20 w-20 object-cover"
        />
      {/if}
      <input
        type="text"
        bind:value={appState.company.name}
        class="font-bold text-2xl focus:outline-none"
        placeholder="Enter Company Name"
      />
    </div>

    <!-- Invoice Title and Number -->
    <div class="text-center sm:text-right mt-4 sm:mt-0">
      <div class="flex flex-col sm:flex-row items-center gap-2">
        <span class="font-semibold text-lg">Invoice:</span>
        <input
          type="text"
          bind:value={appState.invoice.title}
          class="text-3xl font-bold focus:outline-none text-center sm:text-left"
          placeholder="Invoice"
        />
        <span class="font-semibold text-lg">#</span>
        <input
          type="text"
          size="10"
          placeholder="#0001"
          maxlength="20"
          bind:value={appState.invoice.number}
          class="p-2 focus:outline-none w-32 text-center border-b border-gray-400"
        />
      </div>
      <div class="mt-2 flex flex-col sm:flex-row justify-end gap-2">
        <div>
          <span class="secondary-text">Created:</span>
          <input
            bind:value={appState.invoice.created}
            type="date"
            class="p-1 focus:outline-none border-b border-gray-400"
          />
        </div>
        <div>
          <span class="secondary-text">Due:</span>
          <input
            bind:value={appState.invoice.due}
            type="date"
            class="p-1 focus:outline-none border-b border-gray-400"
          />
        </div>
      </div>
      <div class="mt-2">
        <select
          bind:value={appState.currency}
          class="p-1 focus:outline-none border border-gray-400 rounded"
        >
          {#each currencyOptions as option}
            <option value={option.code}>
              {option.flag} {option.code} - {option.name}
            </option>
          {/each}
        </select>
      </div>
    </div>
  </div>

  <hr class="border-gray-300" />

  <!-- From & To -->
  <div class="flex flex-col sm:flex-row justify-between gap-4">
    <!-- Sender Info -->
    <div class="flex flex-col gap-2 w-full">
      <h4 class="mb-2 font-bold">Sender Info</h4>
      <textarea
        style="resize: none;"
        placeholder="Enter sender's name and address"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="p-2 focus:outline-none border-b border-gray-400"
      ></textarea>
      <div class="flex flex-col gap-2">
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Email:</label>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Email Address"
          />
        </div>
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Phone:</label>
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Phone Number"
          />
        </div>
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Tax Details:</label>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxDetails}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Tax Name and ID"
          />
        </div>
      </div>
    </div>

    <!-- Recipient Info -->
    <div class="flex flex-col gap-2 w-full">
      <h4 class="mb-2 font-bold">Recipient Info</h4>
      <textarea
        style="resize: none;"
        placeholder="Enter recipient's name and address"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="p-2 focus:outline-none border-b border-gray-400"
      ></textarea>
      <div class="flex flex-col gap-2">
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Email:</label>
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Email Address"
          />
        </div>
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Phone:</label>
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Phone Number"
          />
        </div>
        <div class="flex flex-col sm:flex-row sm:items-center">
          <label class="w-24 secondary-text">Tax Details:</label>
          <input
            type="text"
            bind:value={appState.invoice.toTaxDetails}
            class="p-2 focus:outline-none w-full sm:w-auto border-b border-gray-400"
            placeholder="Enter Tax Name and ID"
          />
        </div>
      </div>
    </div>
  </div>

  <hr class="border-gray-300 my-4" />

  <!-- Item Descriptions -->
  <div class="flex flex-col mt-4">
    <h4 class="mb-2 font-bold">Item Descriptions</h4>
    <div class="flex flex-row bg-gray-100 p-3 border-b border-gray-300">
      <div class="w-10"></div> <!-- Placeholder for delete button -->
      <p class="font-semibold flex-1">Item Name</p>
      <p class="font-semibold w-32 text-right">Unit Price</p>
      <p class="font-semibold w-24 text-right">Quantity</p>
      <p class="font-semibold w-32 text-right">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row items-center border-b border-gray-300">
        <!-- Delete button -->
        <button
          on:click={() => deleteItem(index)}
          class="p-2 text-red-600 hover:text-red-800 transition-colors duration-200"
          title="Delete Item"
        >
          ✕
        </button>
        <!-- Item name -->
        <input
          type="text"
          bind:value={item.desc}
          class="flex-1 p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter item description"
        />
        <!-- Unit Price -->
        <input
          type="number"
          bind:value={item.price}
          class="w-32 p-2 text-right focus:outline-none border-b border-gray-300"
          placeholder="0"
          min="0"
          step="0.01"
        />
        <!-- Quantity -->
        <input
          type="number"
          bind:value={item.quantity}
          class="w-24 p-2 text-right focus:outline-none border-b border-gray-300"
          placeholder="1"
          min="1"
          step="1"
        />
        <!-- Total -->
        <p class="w-32 text-right">
          {getCurrencySymbol(appState.currency)}
          {formatNumber(item.price * item.quantity)}
        </p>
      </div>
    {/each}

    <!-- Discount -->
    <div class="flex flex-row bg-gray-100 p-3 border-t border-gray-300 items-center">
      <p class="flex-1 text-right font-semibold">Discount %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isDiscountEnabled ? 'active' : ''}"
        on:click={() => (isDiscountEnabled = !isDiscountEnabled)}
        title="Toggle Discount"
      ></button>
      <input
        type="number"
        bind:value={appState.discountPercent}
        class="w-32 p-2 text-right focus:outline-none border-b border-gray-300"
        placeholder="0"
        min="0"
        max="100"
        step="0.01"
        disabled={!isDiscountEnabled}
      />
    </div>

    <!-- Tax -->
    <div class="flex flex-row bg-gray-100 p-3 border-t border-gray-300 items-center">
      <p class="flex-1 text-right font-semibold">Tax %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isTaxEnabled ? 'active' : ''}"
        on:click={() => (isTaxEnabled = !isTaxEnabled)}
        title="Toggle Tax"
      ></button>
      <input
        type="number"
        bind:value={appState.taxPercent}
        class="w-32 p-2 text-right focus:outline-none border-b border-gray-300"
        placeholder="0"
        min="0"
        max="100"
        step="0.01"
        disabled={!isTaxEnabled}
      />
    </div>

    <!-- Total Due -->
    <div class="flex flex-row bg-gray-100 p-3 border-t border-gray-300">
      <p class="flex-1 text-right font-bold">Total Due</p>
      <p class="w-32 text-right font-bold">
        {getCurrencySymbol(appState.currency)}
        {formatNumber(totalDue)}
      </p>
    </div>
  </div>

  <!-- New Item Form -->
  <form class="flex flex-col sm:flex-row justify-between gap-2 mt-4 print:hidden">
    <div class="flex items-center">
      <button
        type="button"
        disabled={itemDesc.trim() === ''}
        on:click={() => addItem()}
        class="p-2 rounded-full bg-black text-white disabled:bg-gray-400 disabled:cursor-not-allowed"
        title="Add Item"
      >
        ＋
      </button>
    </div>

    <div class="flex flex-col sm:flex-row gap-2 w-full">
      <input
        id="descBox"
        type="text"
        bind:value={itemDesc}
        placeholder="Enter item description"
        class="p-2 focus:outline-none flex-1 border-b border-gray-300"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        class="w-32 p-2 focus:outline-none text-right border-b border-gray-300"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        class="w-24 p-2 focus:outline-none text-right border-b border-gray-300"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
      <p class="w-32 text-right">
        {getCurrencySymbol(appState.currency)}
        {formatNumber(itemPrice * itemQty)}
      </p>
    </div>
  </form>

  <!-- Add a line and spacing before Payment Info -->
  <hr class="border-gray-300 my-4" />

  <!-- Payment Info -->
  <div class="mt-4">
    <h4 class="mb-2 font-bold">Payment Info</h4>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <input
          type="text"
          maxlength="20"
          bind:value={appState.payment.accountNumber}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter Account Number"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.accountName}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter Account Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.bank}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter Bank Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.ifsc}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter IFSC Code"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.swiftCode}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter SWIFT Code"
        />
      </div>
      <div class="flex flex-col sm:col-span-2">
        <label class="text-sm font-semibold">Note</label>
        <textarea
          bind:value={appState.payment.note}
          class="p-2 focus:outline-none border-b border-gray-300"
          placeholder="Enter any additional notes or payment instructions"
          rows="3"
        ></textarea>
      </div>
    </div>
  </div>

  <!-- Download Button -->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-6 py-3 rounded-lg bg-black text-white font-semibold flex items-center gap-2"
      title="Download Invoice"
    >
      ⬇
      <span>Download Invoice</span>
    </button>
  </div>
</div>

<style>
  /* Global styles */
  :global(body) {
    font-family: 'DotGothic16', monospace;
    background-color: #f5f5f5;
    color: #000;
  }

  .invoice-container {
    background-color: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(5px);
    border: 1px solid rgba(0, 0, 0, 0.05);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    border-radius: 1rem;
  }

  input,
  textarea,
  select {
    border: none;
    border-bottom: 1px solid #aaa;
    background: transparent;
    color: #000;
    font-family: inherit;
    padding: 4px 0;
    transition: border-bottom-color 0.3s;
  }

  input:focus,
  textarea:focus,
  select:focus {
    outline: none;
    border-bottom: 2px solid #000;
  }

  button {
    color: #000;
    background-color: transparent;
    border: none;
    cursor: pointer;
    font-family: inherit;
  }

  button:hover {
    background-color: #000;
    color: #fff;
  }

  .secondary-text {
    color: #555;
    font-size: 0.9rem;
  }

  .border-custom {
    border-color: rgba(0, 0, 0, 0.1);
  }

  .toggle-switch {
    width: 36px;
    height: 18px;
    background-color: #aaa;
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
    background-color: #fff;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #000;
  }

  .toggle-switch.active:before {
    transform: translateX(18px);
  }

  .toggle-switch:hover {
    background-color: inherit;
  }

  .toggle-switch:hover:before {
    background-color: #fff;
  }

  textarea,
  p,
  input {
    word-wrap: break-word;
  }

  @media print {
    @page {
      margin: 0;
    }
    body {
      margin: 0;
    }
    /* Hide unwanted elements */
    .print\:hidden {
      display: none !important;
    }
    /* Adjust invoice container for print */
    .invoice-container {
      box-shadow: none;
      border: none;
      background-color: #fff;
      backdrop-filter: none;
    }
  }
</style>
