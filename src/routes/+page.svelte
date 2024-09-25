<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte'; // Modern icons
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '',
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
      swiftCode: '',
      customField: ''
    },
    currency: 'USD',
    showDiscount: true,
    showTax: true
  };

  let currencyOptions = [
    { code: 'USD', symbol: '$', name: 'US Dollar' },
    { code: 'EUR', symbol: '€', name: 'Euro' },
    { code: 'GBP', symbol: '£', name: 'British Pound' },
    { code: 'JPY', symbol: '¥', name: 'Japanese Yen' },
    { code: 'AUD', symbol: 'A$', name: 'Australian Dollar' },
    { code: 'CAD', symbol: 'C$', name: 'Canadian Dollar' },
    { code: 'CHF', symbol: 'CHF', name: 'Swiss Franc' },
    { code: 'CNY', symbol: '¥', name: 'Chinese Yuan' },
    { code: 'INR', symbol: '₹', name: 'Indian Rupee' },
    { code: 'RUB', symbol: '₽', name: 'Russian Ruble' },
    { code: 'BRL', symbol: 'R$', name: 'Brazilian Real' },
    { code: 'ZAR', symbol: 'R', name: 'South African Rand' },
    { code: 'MXN', symbol: '$', name: 'Mexican Peso' },
    { code: 'NZD', symbol: 'NZ$', name: 'New Zealand Dollar' },
    { code: 'SGD', symbol: 'S$', name: 'Singapore Dollar' }
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
      appState.items = [...appState.items, { desc: itemDesc, price: itemPrice, quantity: itemQty }];

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
      company: { name: '', logo: '' },
      invoice: {
        number: '',
        created: '',
        due: '',
        from: '',
        fromTaxIdName: 'GSTIN',
        fromTaxId: '',
        fromContact: { mail: '', phone: '' },
        to: '',
        toTaxIdName: 'Interdimensional Tax ID',
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
        swiftCode: '',
        customField: ''
      },
      currency: 'USD',
      showDiscount: true,
      showTax: true
    };

    save();
  }

  // Startup ------------------------------------------------------------

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

  $: subTotal = appState.items.reduce((total, item) => {
    return total + item.price * item.quantity;
  }, 0);

  $: discountAmount = appState.showDiscount
    ? (subTotal * (+appState.discountPercent || 0)) / 100
    : 0;
  $: taxableAmount = subTotal - discountAmount;
  $: taxAmount = appState.showTax ? (taxableAmount * (+appState.taxPercent || 0)) / 100 : 0;
  $: totalDue = taxableAmount + taxAmount;
</script>

<svelte:head>
  <title>Billie - No Strings Attached Invoice Generator</title>
  <meta
    name="description"
    content="Create and download invoices instantly, with zero signups or tracking."
  />
</svelte:head>

<svelte:body on:click={() => save()} />

<div class="app-container max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-inter bg-[#FAFAFA] border border-[#E2E2E2] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none">
  <!-- Reload Button Outside Main Invoice App -->
  <div class="flex justify-end print:hidden">
    <button
      on:click={() => reset()}
      class="reload-button p-2 rounded-lg bg-[#E2E2E2] hover:bg-[#CFCFCF] transition-all duration-100 ease-in-out"
    >
      <RotateCw strokeWidth={1.5} />
    </button>
  </div>

  <!-- Company & Invoice Details -->
  <div class="flex flex-row justify-between items-start">
    <div class="flex flex-col gap-4">
      <div>
        {#if appState.company.logo}
          <img src={appState.company.logo} alt="Company Logo" class="max-h-18 max-w-40 object-cover m-0 print:block" />
        {/if}
        <input
          class="font-bold text-xl border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          type="text"
          bind:value={appState.company.name}
          placeholder="Enter Company Name"
        />
      </div>

      <div class="flex flex-col gap-1 print:hidden">
        <p>
          Created:
          <input
            bind:value={appState.invoice.created}
            type="text"
            size="10"
            placeholder="Date Created"
            maxlength="13"
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          />
        </p>
        <p>
          Due:
          <input
            bind:value={appState.invoice.due}
            type="text"
            size="10"
            placeholder="Due Date"
            maxlength="13"
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          />
        </p>
      </div>
      <select
        bind:value={appState.currency}
        class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A] print:hidden"
      >
        {#each currencyOptions as option}
          <option value={option.code}>{option.code} - {option.name}</option>
        {/each}
      </select>
    </div>
  </div>

  <hr class="border-[#E2E2E2] print:hidden" />

  <!-- From & To -->
  <div class="flex flex-row justify-between">
    <div>
      <h4 class="mb-2 font-bold">From</h4>
      <textarea
        style="resize: none;"
        placeholder="Enter sender's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.from}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="mt-2">
        <p>
          Tax Name:
          <input
            type="text"
            bind:value={appState.invoice.fromTaxIdName}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Tax Name"
          />
        </p>
        <p>
          Tax ID:
          <input
            type="text"
            bind:value={appState.invoice.fromTaxId}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Tax ID"
          />
        </p>
        <p>
          Email:
          <input
            type="email"
            bind:value={appState.invoice.fromContact.mail}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Email"
          />
        </p>
        <p>
          Phone:
          <input
            type="text"
            bind:value={appState.invoice.fromContact.phone}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Phone Number"
          />
        </p>
      </div>
    </div>
    <div>
      <h4 class="mb-2 font-bold">To</h4>
      <textarea
        style="resize: none;"
        placeholder="Enter recipient's address"
        cols="30"
        rows="3"
        maxlength="150"
        bind:value={appState.invoice.to}
        class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
      ></textarea>
      <div class="mt-2">
        <p>
          Tax Name:
          <input
            type="text"
            bind:value={appState.invoice.toTaxIdName}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Tax Name"
          />
        </p>
        <p>
          Tax ID:
          <input
            type="text"
            bind:value={appState.invoice.toTaxId}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Tax ID"
          />
        </p>
        <p>
          Email:
          <input
            type="email"
            bind:value={appState.invoice.toContact.mail}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Email"
          />
        </p>
        <p>
          Phone:
          <input
            type="text"
            bind:value={appState.invoice.toContact.phone}
            class="border border-[#E2E2E2] p-1 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            placeholder="Enter Phone Number"
          />
        </p>
      </div>
    </div>
  </div>

  <!-- Toggles for Discount and Tax -->
  <div class="flex flex-row justify-between items-center py-4 print:hidden">
    <div class="flex items-center">
      <label class="text-sm font-semibold mr-2">Discount</label>
      <input type="checkbox" bind:checked={appState.showDiscount} class="toggle-switch" />
    </div>
    <div class="flex items-center">
      <label class="text-sm font-semibold mr-2">Tax</label>
      <input type="checkbox" bind:checked={appState.showTax} class="toggle-switch" />
    </div>
  </div>

  <!-- Table and Item Management -->
  {#if appState.items.length > 0}
    <div class="flex flex-col">
      <div class="flex flex-row bg-[#FAFAFA] border border-[#E2E2E2]">
        <p class="font-bold p-2 border-r border-[#E2E2E2] grow">Item Description</p>
        <p class="font-bold p-2 border-r border-[#E2E2E2] w-32">Unit Price</p>
        <p class="font-bold p-2 border-r border-[#E2E2E2] w-24">Qty</p>
        <p class="font-bold p-2 w-32 text-right">Total</p>
      </div>
      {#each appState.items as item, index}
        <div class="flex flex-row even:bg-[#F8F8F8] border border-[#E2E2E2]">
          <button
            on:click={() => deleteItem(index)}
            class="p-2 hover:bg-[#E2E2E2] transition-all duration-100 ease-in-out rounded print:hidden"
          >
            <Trash color="#C96868" />
          </button>

          <p contenteditable="true" class="p-2 border-r border-[#E2E2E2] grow">{item.desc}</p>
          <input
            class="p-2 border-r border-[#E2E2E2] w-32 text-right"
            type="number"
            step="0.01"
            min="0"
            max="999999.99"
            bind:value={item.price}
          />
          <input
            class="p-2 border-r border-[#E2E2E2] w-24 text-right"
            type="number"
            step="0.01"
            min="0"
            max="9999.99"
            bind:value={item.quantity}
          />
          <p class="p-2 w-32 text-right">{getCurrencySymbol(appState.currency)}{(item.price * item.quantity).toFixed(2)}</p>
        </div>
      {/each}

      <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
        <p class="p-2 grow text-right font-bold">Subtotal</p>
        <p class="p-2 w-32 text-right">{getCurrencySymbol(appState.currency)}{subTotal.toFixed(2)}</p>
      </div>
      {#if appState.showDiscount}
        <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
          <p class="p-2 grow text-right font-bold">Discount %</p>
          <input class="p-2 w-32 text-right" type="number" step="0.01" min="0" max="100" bind:value={appState.discountPercent} />
        </div>
      {/if}
      <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
        <p class="p-2 grow text-right font-bold">Taxable Amount</p>
        <p class="p-2 w-32 text-right">{getCurrencySymbol(appState.currency)}{taxableAmount.toFixed(2)}</p>
      </div>
      {#if appState.showTax}
        <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
          <p class="p-2 grow text-right font-bold">Tax %</p>
          <input class="p-2 w-32 text-right" type="number" step="0.01" min="0" max="100" bind:value={appState.taxPercent} />
        </div>
      {/if}
      <div class="flex flex-row even:bg-[#F8F8F8] border-t border-[#E2E2E2]">
        <p class="p-2 grow text-right font-bold">Total Due</p>
        <p class="p-2 w-32 text-right">{getCurrencySymbol(appState.currency)}{totalDue.toFixed(2)}</p>
      </div>
    </div>
  {/if}

  <!-- Download Button -->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-2 rounded-lg bg-[#E2E2E2] text-[#333] font-semibold flex items-center gap-2 hover:bg-[#CFCFCF] transition-transform"
    >
      <Download class="w-5 h-5" />
      <span>Download Invoice</span>
    </button>
  </div>
</div>
