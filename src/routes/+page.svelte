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
  <!-- Item Descriptions -->
  <div class="flex flex-col mt-4 p-4 rounded-lg border-2 border-gray-600 space-y-2">
    <h2 class="mb-2 font-bold">Item Descriptions</h2>
    <div class="flex flex-row bg-gray-200 border-2 border-gray-600">
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/2 text-right">Item Name</p>
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/6 text-right">Unit Price</p>
      <p class="font-bold p-3 border-r-2 border-gray-600 w-1/6 text-right">Qty</p>
      <p class="font-bold p-3 w-1/6 text-right">Total</p>
    </div>
    {#each appState.items as item, index}
      <div class="flex flex-row items-center border-2 border-gray-600">
        <button
          on:click={() => deleteItem(index)}
          class="p-3 hover:bg-[#E2E2E2] transition-all duration-100 ease-in-out rounded print:hidden"
        >
          <Trash />
        </button>
        <textarea
          class="p-3 border-r-2 border-gray-600 w-1/2 resize-none"
          bind:value={item.desc}
          rows="1"
          on:input={(e) => { autoExpand(e.target); save(); }}
        ></textarea>
        <input
          class="p-3 border-r-2 border-gray-600 w-1/6 text-right"
          type="number"
          min="0"
          step="0.01"
          bind:value={item.price}
          on:input={() => save()}
        />
        <input
          class="p-3 border-r-2 border-gray-600 w-1/6 text-right"
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

    <!-- Tax, Discount, Total -->
    <div class="flex flex-row items-center border-t-2 border-gray-600 pt-2">
      <p class="p-3 grow text-right font-bold">Discount %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isDiscountEnabled ? 'active' : ''}"
        on:click={() => (isDiscountEnabled = !isDiscountEnabled)}
      ></button>
      <input
        class="p-3 w-32 text-right {isDiscountEnabled ? 'enabled' : 'disabled'}"
        type="number"
        bind:value={appState.discountPercent}
        disabled={!isDiscountEnabled}
        step="0.01"
        max="100"
        min="0"
        on:input={() => save()}
      />
    </div>
    <div class="flex flex-row items-center border-t-2 border-gray-600 pt-2">
      <p class="p-3 grow text-right font-bold">Tax %</p>
      <button
        class="toggle-switch ml-2 mr-1 {isTaxEnabled ? 'active' : ''}"
        on:click={() => (isTaxEnabled = !isTaxEnabled)}
      ></button>
      <input
        class="p-3 w-32 text-right {isTaxEnabled ? 'enabled' : 'disabled'}"
        type="number"
        bind:value={appState.taxPercent}
        disabled={!isTaxEnabled}
        step="0.01"
        max="100"
        min="0"
        on:input={() => save()}
      />
    </div>
    <div class="flex flex-row border-t-2 border-gray-600 pt-2">
      <p class="p-3 grow text-right font-bold">Total Due</p>
      <p class="p-3 w-32 text-right bg-green-100 font-bold">
        {getCurrencySymbol(appState.currency)}{totalDue}
      </p>
    </div>
  </div>

  <!-- Payment Info -->
  <div class="mt-8">
    <h2 class="mb-4 font-bold">Payment Info</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account No</label>
        <textarea
          bind:value={appState.payment.accountNumber}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] resize-none"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Account Name</label>
        <textarea
          bind:value={appState.payment.accountName}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] resize-none"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">Bank Name</label>
        <textarea
          bind:value={appState.payment.bank}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] resize-none"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">IFSC</label>
        <textarea
          bind:value={appState.payment.ifsc}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] resize-none"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
      <div class="flex flex-col">
        <label class="text-sm font-semibold">SWIFT Code</label>
        <textarea
          bind:value={appState.payment.swiftCode}
          class="border-2 border-gray-600 p-2 rounded-lg focus:outline-none focus:border-[#5A5A5A] resize-none"
          on:input={(e) => autoExpand(e.target)}
          rows="1"
        ></textarea>
      </div>
    </div>
  </div>

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
