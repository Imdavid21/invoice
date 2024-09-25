<script>
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      title: 'Invoice',
      serialLabel: 'Serial No.',
      serialNumber: '#001',
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
        serialLabel: 'Serial No.',
        serialNumber: '#001',
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

  // Handle responsive design
  function handleResize() {
    const width = window.innerWidth;
    isMobile = width < 640; // Assuming 640px as the breakpoint for mobile devices
  }

  let isMobile = false;

  onMount(() => {
    handleResize();
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
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
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-4 py-8 flex flex-col space-y-6 invoice-container print:shadow-none print:bg-white print:border-none"
>
  <!-- INVOICE Title and Serial No. at Top Center -->
  <div class="text-center mb-6">
    <h2 class="font-bold text-3xl">
      <input
        type="text"
        bind:value={appState.invoice.title}
        class="text-center p-2 focus:outline-none w-auto"
        placeholder="Invoice"
      />
    </h2>
    <div class="mt-2">
      <div class="flex flex-col justify-center items-center gap-2">
        <label class="text-lg font-semibold">
          {appState.invoice.serialLabel}:
        </label>
        <input
          type="text"
          bind:value={appState.invoice.serialNumber}
          class="p-2 focus:outline-none w-32 text-center border-b-2 border-transparent focus:border-black"
          placeholder="#001"
        />
      </div>
    </div>
    <div class="mt-4"></div>
  </div>

  <!-- Company & Invoice Details -->
  <div class="flex flex-col justify-between items-start print:flex">
    <div class="flex flex-col gap-4 w-full">
      <div class="">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-row print:hidden">
              <button
                class="p-2 hover:bg-gray-700 hover:text-white rounded transition-colors duration-150"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                ✕
              </button>
              <button
                class="p-2 hover:bg-gray-700 hover:text-white rounded transition-colors duration-150 ml-2"
                on:click={() => document.getElementById('imageInput').click()}
              >
                ＋
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
            class="p-2 flex flex-row gap-2 rounded-lg cursor-pointer hover:bg-gray-700 hover:text-white transition-colors duration-150 print:hidden"
            on:click={() => document.getElementById('imageInput').click()}
          >
            ＋
            <p class="text-sm">Add Logo</p>
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
        type="text"
        bind:value={appState.company.name}
        class="font-bold text-xl p-2 focus:outline-none w-full"
        placeholder="Enter Company Name"
      />
    </div>

    <div class="relative flex flex-col items-start gap-2 mt-4 w-full">
      <div class="flex flex-col gap-1 w-full">
        <p class="flex items-center justify-between">
          <span class="mr-2 secondary-text">Created :</span>
          <input
            bind:value={appState.invoice.created}
            type="date"
            class="p-2 focus:outline-none"
          />
        </p>
        <p class="flex items-center justify-between">
          <span class="mr-2 secondary-text">Due :</span>
          <input
            bind:value={appState.invoice.due}
            type="date"
            class="p-2 focus:outline-none"
          />
        </p>
      </div>
      <select
        bind:value={appState.currency}
        class="p-2 focus:outline-none w-full mt-2"
      >
        {#each currencyOptions as option}
          <option value={option.code}>
            {option.flag} {option.code} - {option.name}
          </option>
        {/each}
      </select>
    </div>
  </div>

  <hr class="border-custom" />

  <!-- From & To -->
  <div class="flex flex-col justify-between gap-4">
    <!-- Sender Info -->
    <div class="flex flex-col gap-2 w-full">
      <h4 class="mb-2 font-bold">Sender Info</h4>
      <textarea
        style="resize: none; padding: 8px 0;"
        placeholder="Enter sender's name and address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="p-2 focus:outline-none w-full break-words"
      ></textarea>
      <div class="flex flex-col gap-2">
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Email Address"
          />
        </div>
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Phone Number"
          />
        </div>
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Tax Details:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxDetails}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Tax Name and ID"
          />
        </div>
      </div>
    </div>
    <!-- Recipient Info -->
    <div class="flex flex-col gap-2 w-full mt-4">
      <h4 class="mb-2 font-bold">Recipient Info</h4>
      <textarea
        style="resize: none; padding: 8px 0;"
        placeholder="Enter recipient's name and address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="p-2 focus:outline-none w-full break-words"
      ></textarea>
      <div class="flex flex-col gap-2">
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Email:</span>
          <input
            type="email"
bind:value={appState.invoice.toContact.mail}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Email Address"
          />
        </div>
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Phone Number"
          />
        </div>
        <div class="flex flex-col items-start">
          <span class="w-full secondary-text">Tax Details:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxDetails}
            class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
            placeholder="Enter Tax Name and ID"
          />
        </div>
      </div>
    </div>
  </div>

  <!-- Item Descriptions -->
  <div class="flex flex-col mt-4">
    <!-- Table Headers -->
    <div class="flex flex-col border-b border-custom items-start">
      <div class="p-3 w-full flex justify-between">
        <p class="font-bold">Item Name</p>
        <p class="font-bold">Unit Price</p>
      </div>
      <div class="p-3 w-full flex justify-between">
        <p class="font-bold">Quantity</p>
        <p class="font-bold">Total</p>
      </div>
    </div>
    
    <!-- Item Rows -->
    {#each appState.items as item, index}
      <div class="flex flex-col items-start border-b border-custom">
        <div class="p-3 w-full flex justify-between items-center">
          <input
            type="text"
            bind:value={item.desc}
            class="grow break-words border-none focus:ring-0"
            placeholder="Enter Item Name"
          />
          <input
            type="number"
            bind:value={item.price}
            class="w-24 text-right border-b-2 border-transparent focus:border-black focus:outline-none"
            placeholder="0"
            min="0"
            step="0.01"
          />
        </div>
        <div class="p-3 w-full flex justify-between items-center">
          <input
            type="number"
            bind:value={item.quantity}
            class="w-24 text-left border-b-2 border-transparent focus:border-black focus:outline-none"
            placeholder="0"
            min="1"
            step="1"
          />
          <p class="w-24 text-right">
            {getCurrencySymbol(appState.currency)}
            {formatNumber(item.price * item.quantity)}
          </p>
        </div>
        <button
          on:click={() => deleteItem(index)}
          class="p-3 hover:bg-gray-700 hover:text-white transition-colors duration-150 rounded print:hidden w-full text-center"
        >
          Delete Item
        </button>
      </div>
    {/each}

    <!-- Discount, Tax, Total Due Rows -->
    <div class="flex flex-col border-t border-custom items-start" style="background-color: #F8F8F8;">
      <div class="p-3 w-full flex justify-between items-center">
        <p class="font-bold">Discount %</p>
        <div class="flex items-center">
          <button class="toggle-switch mr-2 {isDiscountEnabled ? 'active' : ''}" on:click={() => (isDiscountEnabled = !isDiscountEnabled)}></button>
          <input
            class="w-24 text-right border-b-2 border-transparent focus:border-black focus:outline-none"
            type="number"
            bind:value={appState.discountPercent}
            disabled={!isDiscountEnabled}
            step="0.01"
            max="100"
            min="0"
            placeholder="0"
          />
        </div>
      </div>
    </div>
    <div class="flex flex-col border-t border-custom items-start" style="background-color: #F8F8F8;">
      <div class="p-3 w-full flex justify-between items-center">
        <p class="font-bold">Tax %</p>
        <div class="flex items-center">
          <button class="toggle-switch mr-2 {isTaxEnabled ? 'active' : ''}" on:click={() => (isTaxEnabled = !isTaxEnabled)}></button>
          <input
            class="w-24 text-right border-b-2 border-transparent focus:border-black focus:outline-none"
            type="number"
            bind:value={appState.taxPercent}
            disabled={!isTaxEnabled}
            step="0.01"
            max="100"
            min="0"
            placeholder="0"
          />
        </div>
      </div>
    </div>
    <div class="flex flex-col border-t border-custom" style="background-color: #F8F8F8;">
      <div class="p-3 w-full flex justify-between items-center">
        <p class="font-bold">Total Due</p>
        <p class="w-24 text-right">
          {getCurrencySymbol(appState.currency)}{formatNumber(totalDue)}
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form -->
  <form class="flex flex-col justify-between gap-2 mt-4 print:hidden">
    <div class="flex items-center justify-between w-full">
      <input
        id="descBox"
        type="text"
        bind:value={itemDesc}
        placeholder="Enter item description"
        class="p-2 focus:outline-none grow border-b-2 border-transparent focus:border-black"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
    </div>
    <div class="flex items-center justify-between w-full mt-2">
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        class="p-2 focus:outline-none w-1/2 border-b-2 border-transparent focus:border-black"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        class="p-2 focus:outline-none w-1/2 border-b-2 border-transparent focus:border-black"
        on:keypress={(e) => e.key === 'Enter' && addItem()}
      />
    </div>
    <button
      type="button"
      on:click={addItem}
      class="p-2 mt-2 bg-black text-white hover:bg-gray-700 transition-colors duration-150 rounded"
    >
      Add Item
    </button>
  </form>

  <!-- Payment Info -->
  <div class="mt-6">
    <h4 class="mb-4 font-bold text-2xl">Payment Info</h4>
    <div class="grid grid-cols-1 gap-6">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <input
          type="text"
          maxlength="20"
          bind:value={appState.payment.accountNumber}
          class="p-2 focus:outline-none border-b-2 border-transparent focus:border-black"
          placeholder="Enter Account Number"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.accountName}
          class="p-2 focus:outline-none border-b-2 border-transparent focus:border-black"
          placeholder="Enter Account Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.bank}
          class="p-2 focus:outline-none border-b-2 border-transparent focus:border-black"
          placeholder="Enter Bank Name"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.ifsc}
          class="p-2 focus:outline-none border-b-2 border-transparent focus:border-black"
          placeholder="Enter IFSC Code"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.swiftCode}
          class="p-2 focus:outline-none border-b-2 border-transparent focus:border-black"
          placeholder="Enter SWIFT Code"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Note</label>
        <textarea
          bind:value={appState.payment.note}
          class="p-2 focus:outline-none w-full border-b-2 border-transparent focus:border-black"
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
      class="w-full px-6 py-3 rounded-lg bg-black text-white font-semibold flex items-center justify-center gap-2 hover:bg-gray-700 transition-colors duration-150"
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
    margin: 0;
    padding: 0;
  }

  .invoice-container {
    background-color: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
    border-radius: 1rem;
    padding: 2rem;
  }

  input,
  textarea,
  select {
    border: none;
    border-bottom: 1px solid #aaa;
    background: transparent;
    color: #000;
    font-family: inherit;
    padding: 8px 0;
    transition: border-color 0.3s;
  }

  input:focus,
  textarea:focus,
  select:focus {
    outline: none;
    border-bottom: 1px solid #000;
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
  }

  .border-custom {
    border-color: rgba(0, 0, 0, 0.1);
  }

  .toggle-switch {
    width: 40px;
    height: 20px;
    background-color: #aaa;
    border-radius: 20px;
    position: relative;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .toggle-switch:before {
    content: '';
    position: absolute;
    top: 2px;
    left: 2px;
    width: 16px;
    height: 16px;
    background-color: #fff;
    border-radius: 50%;
    transition: all 0.3s;
  }

  .toggle-switch.active {
    background-color: #4ade80;
  }

  .toggle-switch.active:before {
    transform: translateX(20px);
  }

  .toggle-switch:hover {
    background-color: #777;
  }

  .toggle-switch:hover:before {
    background-color: #fff;
  }

  textarea,
  p,
  input {
    word-wrap: break-word;
  }

  /* Print Styles */
  @media print {
    @page {
      margin: 0;
    }
    body {
      margin: 0;
    }
    .print\:hidden {
      display: none !important;
    }
    .invoice-container img {
      display: block;
    }
  }
</style>
