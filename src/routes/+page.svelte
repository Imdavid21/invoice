<script>
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
    items: [{ desc: '', price: '', quantity: '' }],
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

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
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
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-col sm:flex-row justify-between items-start print:flex">
    <div class="flex flex-col gap-4 w-full sm:w-auto">
      <div class="">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-col">
              <button
                class="p-2 hover:bg-[#E2E2E2] rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                ✕
              </button>
              <button
                class="p-2 hover:bg-[#E2E2E2] rounded"
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
            class="p-2 flex flex-row gap-2 rounded-lg cursor-pointer hover:bg-[#F5F5F5]"
            on:click={() => document.getElementById('imageInput').click()}
          >
            ＋
            <p class="text-sm text-[#333]">Click to select an image for logo</p>
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
        placeholder="Company Name"
      />
    </div>

    <div class="relative flex flex-col items-start sm:items-end gap-2 mt-4 sm:mt-0 w-full sm:w-auto">
      <h2 class="font-bold text-2xl text-black text-left sm:text-right print:text-left">
        INVOICE :
        <input
          type="text"
          size="4"
          placeholder="#0001"
          maxlength="5"
          bind:value={appState.invoice.number}
          class="p-2 focus:outline-none w-20"
        />
      </h2>
      <div class="flex flex-col gap-1">
        <p class="flex items-center">
          <span class="mr-2 secondary-text">Created :</span>
          <input
            bind:value={appState.invoice.created}
            type="date"
            class="p-2 focus:outline-none"
          />
        </p>
        <p class="flex items-center">
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
        class="p-2 focus:outline-none w-full"
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

  <!-- From & To  ------------------------------------------------------>
  <div class="flex flex-col sm:flex-row justify-between gap-4">
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
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24 secondary-text">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
    <div class="flex flex-col gap-2 w-full">
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
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-2">
        <p class="flex items-center">
          <span class="w-24 secondary-text">Tax Name:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Tax Name"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Tax ID:</span>
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Tax ID"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Email:</span>
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Email"
          />
        </p>
        <p class="flex items-center">
          <span class="w-24 secondary-text">Phone:</span>
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="p-2 focus:outline-none w-full"
            placeholder="Enter Phone"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- New Item Form --------------------------------------------------->
  <form class="flex flex-col sm:flex-row justify-between gap-2">
    <div class="flex items-center">
      <button
        disabled={itemDesc == ''}
        on:click={() => addItem()}
        class="p-2 rounded-full bg-transparent hover:bg-black text-black hover:text-white transition-all duration-100 ease-in-out print:hidden"
      >
        ＋
      </button>
    </div>

    <div class="flex flex-col sm:flex-row gap-2 w-full">
      <input
        id="descBox"
        type="text"
        bind:value={itemDesc}
        placeholder="Item name"
        class="p-2 focus:outline-none grow w-full"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemPrice}
        placeholder="Unit Price"
        min="0"
        step="0.01"
        class="p-2 focus:outline-none w-full sm:w-32"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
      />
      <input
        type="number"
        bind:value={itemQty}
        placeholder="Quantity"
        min="1"
        step="1"
        class="p-2 focus:outline-none w-full sm:w-32"
        on:keypress={(e) => e.key == 'Enter' && addItem()}
      />
      <p class="p-2 w-full sm:w-32 text-right">
        {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
      </p>
    </div>
  </form>

  <!-- Table ----------------------------------------------------------->
  <div class="flex flex-col mt-4">
    <h4 class="mb-2 font-bold">Item Descriptions</h4>
    <div class="flex flex-row border-b border-custom">
      <p class="font-bold p-3 grow">Item name</p>
      <p class="font-bold p-3 w-32 hidden sm:block">Unit Price</p>
      <p class="font-bold p-3 w-24 hidden sm:block">Qty</p>
      <p class="font-bold p-3 w-32 text-right hidden sm:block">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-col sm:flex-row even:bg-[#F8F8F8] border-b border-custom">
        <div class="flex flex-row items-center">
          <button
            on:click={() => deleteItem(index)}
            class="p-3 hover:bg-black hover:text-white transition-all duration-100 ease-in-out rounded print:hidden"
          >
            ✕
          </button>
          <p contenteditable="true" class="p-3 grow break-words">{item.desc}</p>
        </div>
        <div class="flex flex-row sm:flex-row w-full">
          <input class="p-3 w-full sm:w-32" type="text" bind:value={item.price} />
          <input class="p-3 w-full sm:w-24" type="text" bind:value={item.quantity} />
          <p class="p-3 w-full sm:w-32 text-right">{getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}</p>
        </div>
      </div>
    {/each}

    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-custom items-center">
      <p class="p-3 grow text-right font-bold">Discount %</p>
      <button class="toggle-switch ml-2 mr-1 {isDiscountEnabled ? 'active' : ''}" on:click={() => (isDiscountEnabled = !isDiscountEnabled)}></button>
      <input
        class="p-3 w-32 text-right"
        type="text"
        bind:value={appState.discountPercent}
        disabled={!isDiscountEnabled}
        step="0.01"
        max="100"
        min="0"
      />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-custom items-center">
      <p class="p-3 grow text-right font-bold">Tax %</p>
      <button class="toggle-switch ml-2 mr-1 {isTaxEnabled ? 'active' : ''}" on:click={() => (isTaxEnabled = !isTaxEnabled)}></button>
      <input class="p-3 w-32 text-right" type="text" bind:value={appState.taxPercent} disabled={!isTaxEnabled} step="0.01" max="100" min="0" />
    </div>
    <div class="flex flex-row even:bg-[#F8F8F8] border-t border-custom">
      <p class="p-3 grow text-right font-bold">Total Due</p>
      <p class="p-3 w-32 text-right">{getCurrencySymbol(appState.currency)}{totalDue}</p>
    </div>
  </div>

  <!-- Payment Info Section Redesigned --------------------------------->
  <div class="mt-4">
    <h4 class="mb-2 font-bold">Payment Info</h4>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <input
          type="text"
          maxlength="20"
          bind:value={appState.payment.accountNumber}
          class="p-2 focus:outline-none"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.accountName}
          class="p-2 focus:outline-none"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <input
          type="text"
          maxlength="28"
          bind:value={appState.payment.bank}
          class="p-2 focus:outline-none"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.ifsc}
          class="p-2 focus:outline-none"
        />
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <input
          type="text"
          maxlength="11"
          bind:value={appState.payment.swiftCode}
          class="p-2 focus:outline-none"
        />
      </div>
    </div>
  </div>

  <!-- Download Button -------------------------------------------------->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-transparent text-black font-semibold flex items-center gap-2 hover:bg-black hover:text-white transition-transform duration-150"
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
    background-color: rgba(255, 255, 255, 0.85);
    backdrop-filter: blur(10px);
    border: 1px solid rgba(0, 0, 0, 0.1);
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
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
    padding: 8px 0;
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

  textarea,
  p,
  input {
    word-wrap: break-word;
  }
</style>
