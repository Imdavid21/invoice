<script>
  import { Plus, Trash, ImagePlus, RotateCw, Download } from 'lucide-svelte';
  import { onMount } from 'svelte';

  let appState = {
    company: { name: '', logo: '' },
    invoice: {
      number: '007',
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
    paymentFields: [
      { label: 'Account No.', value: '' },
      { label: 'Account Name', value: '' },
      { label: 'Bank Name', value: '' },
      { label: 'IFSC', value: '' },
      { label: 'SWIFT Code', value: '' }
    ],
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

  // Utils --------------------------------------------------------------

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
    // Function intentionally left blank to prevent localStorage usage.
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
        number: '007',
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
        { desc: 'Batmobile Repair', price: '15000', quantity: '1' },
        { desc: 'Grappling Hooks', price: '500', quantity: '20' },
        { desc: 'Interdimensional Portal Gun', price: '200000', quantity: '1' }
      ],
      taxPercent: '',
      discountPercent: '',
      note: '',
      paymentFields: [
        { label: 'Account No.', value: '' },
        { label: 'Account Name', value: '' },
        { label: 'Bank Name', value: '' },
        { label: 'IFSC', value: '' },
        { label: 'SWIFT Code', value: '' }
      ],
      currency: 'USD'
    };

    save();
  }

  function addPaymentField() {
    appState.paymentFields.push({ label: 'New Field', value: '' });
    save();
  }

  function removePaymentField(index) {
    appState.paymentFields.splice(index, 1);
    save();
  }

  // Startup ------------------------------------------------------------

  onMount(() => {
    reset();
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
</svelte:head>

<svelte:body on:click={() => save()} />

<div
  class="max-w-screen-md mx-auto px-6 py-8 flex flex-col space-y-6 font-montserrat bg-[#FAFAFA] border border-[#E2E2E2] rounded-xl shadow-sm print:shadow-none print:bg-white print:border-none"
>
  <!-- Company & Invoice Details --------------------------------------->
  <div class="flex flex-col space-y-4 print:flex print:flex-col">
    <div class="flex flex-row justify-between items-start">
      <div class="flex flex-col gap-1">
        {#if appState.company.logo}
          <div class="flex flex-row items-center space-x-4">
            <div class="flex flex-col border border-[#E2E2E2] rounded-xl">
              <button
                class="p-2 hover:bg-[#E2E2E2] rounded"
                on:click={() => {
                  appState.company.logo = null;
                }}
              >
                <Trash />
              </button>
              <button
                class="p-2 hover:bg-[#E2E2E2] rounded"
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
            class="p-2 flex flex-row gap-2 rounded-lg border border-[#E2E2E2] cursor-pointer hover:bg-[#F5F5F5]"
            on:click={() => document.getElementById('imageInput').click()}
          >
            <ImagePlus />
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
        <input
          class="font-bold text-xl border border-[#E2E2E2] p-3 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          type="text"
          bind:value={appState.company.name}
        />
      </div>

      <div class="relative flex flex-col items-end gap-1 print:items-start print:w-full">
        <h2 class="font-bold text-2xl text-[#1E6F5C] text-right print:text-left">
          INVOICE :
          <input
            type="text"
            size="4"
            placeholder="007"
            maxlength="4"
            bind:value={appState.invoice.number}
            class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          />
        </h2>
        <button
          on:click={() => reset()}
          class="p-2 rounded-lg hover:bg-[#E2E2E2] transition-all duration-100 ease-in-out print:hidden"
        >
          <RotateCw strokeWidth={1.5} />
        </button>
        <div class="flex flex-row gap-4 items-center">
          <div class="flex flex-col">
            <label class="text-sm font-semibold">Currency</label>
            <select
              bind:value={appState.currency}
              class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            >
              {#each currencyOptions as option}
                <option value={option.code}>
                  {option.flag} {option.name}
                </option>
              {/each}
            </select>
          </div>
          <div class="flex flex-col">
            <label class="text-sm font-semibold">Invoice Date</label>
            <input
              type="date"
              bind:value={appState.invoice.created}
              class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            />
          </div>
          <div class="flex flex-col">
            <label class="text-sm font-semibold">Due Date</label>
            <input
              type="date"
              bind:value={appState.invoice.due}
              class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
            />
          </div>
        </div>
      </div>
    </div>

    <hr class="border-[#E2E2E2]" />

    <!-- From & To Section -->
    <div class="flex flex-row gap-4">
      <div class="flex flex-col gap-2 w-1/2">
        <h4 class="mb-1 font-bold">From</h4>
        <input
          type="text"
          placeholder="Business Name"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.from}
        />
        <input
          type="email"
          placeholder="name@business.com"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.fromContact.mail}
        />
        <input
          type="text"
          placeholder="Street"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.from}
        />
        <input
          type="text"
          placeholder="(123) 456 789"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.fromContact.phone}
        />
        <input
          type="text"
          placeholder="123-45-6789"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.fromTaxId}
        />
      </div>

      <div class="flex flex-col gap-2 w-1/2">
        <h4 class="mb-1 font-bold">Bill To</h4>
        <input
          type="text"
          placeholder="Client Name"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.to}
        />
        <input
          type="email"
          placeholder="name@client.com"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.toContact.mail}
        />
        <input
          type="text"
          placeholder="Street"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.to}
        />
        <input
          type="text"
          placeholder="(123) 456 789"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.toContact.phone}
        />
        <input
          type="text"
          placeholder="123-45-6789"
          class="border border-[#E2E2E2] p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A]"
          bind:value={appState.invoice.toTaxId}
        />
      </div>
    </div>
  </div>

  <!-- New Item Form -->
  <form class="flex flex-row justify-between gap-2 mt-4">
    <input
      id="descBox"
      type="text"
      bind:value={itemDesc}
      placeholder="Item description"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] grow"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemPrice}
      placeholder="Unit cost"
      min="0"
      step="0.01"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <input
      type="number"
      bind:value={itemQty}
      placeholder="Quantity"
      min="1"
      step="1"
      class="border border-[#E2E2E2] rounded-lg p-3 focus:outline-none focus:border-[#5A5A5A] w-32"
      on:keypress={(e) => e.key == 'Enter' && addItem()}
    />
    <p class="border border-[#E2E2E2] p-3 w-32 text-right">
      {getCurrencySymbol(appState.currency)}{(itemPrice * itemQty).toFixed(2)}
    </p>
    <button
      on:click={() => addItem()}
      class="p-2 rounded-lg bg-[#8EACCD] text-white hover:bg-[#6E9BBE] transition-all duration-100 ease-in-out"
    >
      <Plus />
    </button>
  </form>

  <!-- Download Button -->
  <div class="mt-6 flex justify-center print:hidden">
    <button
      on:click={() => window.print()}
      class="px-4 py-3 rounded-lg bg-[#E2E2E2] text-[#333] font-semibold flex items-center gap-2 hover:bg-[#CFCFCF] transition-transform duration-150"
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
    background-color: #feffd9;
  }
</style>
